# Trixie Gotchas

Raspberry Pi OS 13 (Trixie) — released in 2025 — is a fresh enough
target that several ConsolePi behaviors need a heads-up. Nothing here
is a blocker; all are workarounds documented from a real working build.

## mDNS A record shows the hotspot IP

**Symptom:** `avahi-resolve -n ConsolePi.local` returns `10.110.0.1`
instead of the Pi's actual LAN IP (e.g., `192.168.1.142`).

**Why:** ConsolePi runs its own mDNS registrar (`consolepi-mdnsreg`)
instead of avahi. It picks an interface to advertise, and if the
built-in auto-hotspot's `dnsmasq` is running (which it is by default),
its subnet IP (`10.110.0.1`) can end up as the A record.

The **real** IP is still stashed in the mDNS TXT record. Query it fully:

```bash
avahi-browse -rt _consolepi._tcp
```

You'll see the TXT payload has a `rem_ip` field with the correct IP,
plus a full `interfaces={}` JSON with per-NIC details.

**Workarounds:**

- Just use the IP directly for telnet / SSH — ignore the mDNS A record
- Or disable the auto-hotspot (edit `/etc/ConsolePi/ConsolePi.yaml`, set
  `wired_dhcp: false` and the hotspot bits, and remove the `10.110.0.0/24`
  interface). This is invasive; only worth it if the mDNS quirk really
  bothers you.

**Not affected:** SSH to `ConsolePi.local` normally works fine anyway
because `nss-mdns` on the client side sometimes falls through to unicast
DNS or uses ARP. The advertised A record is only used when strict mDNS
is the only resolution path.

## `consolepi-menu` crashes in non-interactive shells

**Symptom:** Running `consolepi-menu` from a non-interactive SSH session
(`ssh consolepi@ConsolePi.local consolepi-menu`, or via `sudo -u
consolepi bash -c 'consolepi-menu'`) crashes with:

```
File "/etc/ConsolePi/src/pypkg/consolepi/menu.py", line 307,
    in body_avail_rows
    return tty.rows - sum(map(len, parts) or [0]) - 1
TypeError: unsupported operand type(s) for -: 'NoneType' and 'int'
```

**Why:** The menu queries `tty.rows` (terminal row count) to size its
output. In a non-interactive shell, no PTY is allocated and `tty.rows`
returns `None`.

**Workaround:** Use an interactive SSH session (`ssh
consolepi@ConsolePi.local` with no trailing command), or force a PTY
with `ssh -t`:

```bash
ssh -t consolepi@ConsolePi.local consolepi-menu
```

**Not a bug worth fixing upstream** for most users — the menu is meant
for interactive use.

## ser2net stanzas only cover `ttyUSB0-7` and `ttyACM0-7`

**Symptom:** You plug in an 8th or higher-numbered adapter and can't
reach it on telnet.

**Why:** The default `/etc/ser2net.yaml` shipped by ConsolePi's
installer covers `ttyUSB0-7` (ports 8001-8008) and `ttyACM0-7` (ports
9000-9007). Beyond 8 of either kind, you'd need to add stanzas.

**Workaround:** Edit `/etc/ser2net.yaml` and add:

```yaml
connection: &ttyACM8
  accepter: telnet(rfc2217),tcp,9008
  connector: serialdev,/dev/ttyACM8,9600n81,local
  enable: on
  options:
    banner: *banner
    kickolduser: true
    telnet-brk-on-sync: true
```

Then `sudo systemctl restart ser2net`.

For real fleets you probably want a second ConsolePi rather than
scaling one to 12+ adapters.

## `rfcomm` (Bluetooth Console) warning on install

**Symptom:** During the ConsolePi silent install, exactly one warning:

```
[WARNING][Bluetooth Console] rfcomm failed to start,
    may be normal depending on service/hardware
```

**Why:** ConsolePi's installer tries to enable Bluetooth serial console
support. Under Trixie's newer BlueZ, the `rfcomm` unit sometimes fails
to start immediately on install.

**Workaround:** Ignore unless you actually plan to use Bluetooth
console. For a LAN-only console server, `rfcomm` isn't needed.

## NetworkManager instead of `dhcpcd`

**Symptom:** ConsolePi documentation from earlier Raspbian versions
references `/etc/dhcpcd.conf` for static-IP config; that file doesn't
exist on Trixie.

**Why:** Trixie ships with NetworkManager as the default network
manager, not `dhcpcd`.

**Workaround:** Use `nmcli` or NetworkManager config files instead.
See **[First Boot](first-boot.md#set-a-static-ip-optional)** for
examples.

The ConsolePi installer itself handles this correctly on Trixie — this
gotcha only bites you when following older ConsolePi docs.

## Python 3.13 is the default

**Symptom:** None you're likely to hit — but worth knowing.

**Why:** Trixie ships Python 3.13. The ConsolePi installer creates its
own venv, so system Python version isn't a compatibility risk, but if
you install ConsolePi Python dependencies system-wide manually,
double-check they're 3.13-compatible.

## Auto-hotspot may not work correctly

**Symptom:** ConsolePi's auto-hotspot feature (fall back to being a
WiFi AP when no wired network is found) may not switch cleanly under
Trixie's NetworkManager stack.

**Why:** Older ConsolePi versions relied on `dhcpcd` hooks that Trixie
doesn't have.

**Workaround:** If you need the auto-hotspot for a truly-remote
"parachute in and console via WiFi" scenario, test it thoroughly. If
you're using ConsolePi as a rack-mounted wired-only OOB device (which
this guide assumes), the hotspot isn't needed and you can leave it
disabled without impact.

## What's fine on Trixie (despite what older docs say)

- ser2net v4 config in `/etc/ser2net.yaml` — works correctly
- `consolepi-menu` interactive mode — works correctly
- Multi-adapter detection — works correctly
- systemd unit generation — works correctly
- REST API on `:5000` — works correctly
- mDNS advertisement (aside from the A-record quirk above) — works
