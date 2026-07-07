# Troubleshooting

Organized by symptom. If you don't find your issue here, check the
[upstream ConsolePi issues](https://github.com/Pack3tL0ss/ConsolePi/issues).

## SSH: can't reach `ConsolePi.local`

Try the IP directly. Find it from your router's DHCP lease table
(look for hostname `ConsolePi` or Pi MAC OUI: `b8:27:eb`, `dc:a6:32`,
`d8:3a:dd`).

If IP works but hostname doesn't:

- Your OS/network may not support mDNS. On some corporate networks
  mDNS is blocked. Use the IP + a DHCP reservation, or add
  `<pi-ip> consolepi.local` to your `/etc/hosts`.
- Some UDMs / consumer routers cross-VLAN block mDNS by default. Enable
  the mDNS/Bonjour reflector on the UDM if you want cross-VLAN
  discovery.

## Telnet to port 9000 / 8001: "Connection refused"

The port isn't listening. Check ser2net:

```bash
sudo systemctl status ser2net
sudo ss -tlnp | grep ser2net
```

Common causes:

- `ser2net` isn't running: `sudo systemctl restart ser2net`
- `ser2net` config syntax error: `sudo journalctl -u ser2net --since '5 min ago'`
- No adapter attached at that `/dev/tty*` device — ser2net still opens
  the port, so this shouldn't cause "connection refused"; but a config
  issue with the port stanza could

## Telnet connects but shows nothing

- Try hitting `Enter` — many devices' console prints on demand
- Try baud rate mismatch (default is 9600; some devices need 115200):
  ```bash
  ssh pi@ConsolePi.local
  sudo screen /dev/ttyACM0 115200
  ```
- Check the cable — for USB-A→USB-C, is it a data cable or charge-only?
- Check for RJ45 rollover pinout — a regular Ethernet cable won't work
- On dual-console Cisco gear, USB-C wins over RJ45. Try unplugging the
  other cable.

## `/dev/ttyACM0` or `/dev/ttyUSB0` doesn't appear

The USB device isn't enumerating. Check:

```bash
lsusb
sudo dmesg -T | tail -30
```

Common causes:

- Cable is charge-only (USB-C): use a known data cable
- Bad cable / bad port: try another
- Target device is off / booting: wait, or reseat power
- Target hasn't yet initialized console redirection (Cisco UCS servers
  in particular need serial console redirect enabled in BIOS/CIMC)

## `/dev/ttyACM0` appears, then disappears, then reappears

Under-powered Pi PSU is the #1 cause. USB peripherals draw enough to
brown out a marginal PSU. Use the official Pi PSU.

Also possible: a flaky USB cable or a USB hub without its own power supply.

## `consolepi-menu` doesn't launch on login

- Did you install with the `-L` flag? Check with:
  ```bash
  sudo cat /home/consolepi/.bashrc | grep -i consolepi-menu
  ```
- Are you logged in as the `consolepi` user, not `pi`? The auto-launch
  is scoped to `consolepi`.
- Is your shell interactive? Non-interactive SSH sessions won't launch
  the menu. See **[Trixie Gotchas](trixie-gotchas.md#consolepi-menu-crashes-in-non-interactive-shells)**.

## `consolepi-menu` crashes with `TypeError: 'NoneType' and 'int'`

Non-interactive shell. Use `ssh -t` or a real interactive session. See
[Trixie Gotchas](trixie-gotchas.md#consolepi-menu-crashes-in-non-interactive-shells).

## `consolepi-details` shows adapter, but ser2net port 9000 refused

Try restarting ser2net:

```bash
sudo systemctl restart ser2net
sleep 2
sudo ss -tlnp | grep ser2net
```

If a stanza in `/etc/ser2net.yaml` is malformed, ser2net will not open
that particular port. Check:

```bash
sudo journalctl -u ser2net --since '5 min ago' | grep -iE 'error|warning'
```

## Firewall is dropping console output mid-boot

Very rare but possible. If the target device is spitting output faster
than the Pi can drain, ser2net's default buffers may overflow. This
manifests as missing characters, not a full drop.

Fix by increasing rsize/wsize in the ser2net stanza:

```yaml
connection: &ttyACM0
  accepter: telnet(rfc2217),tcp,9000
  connector: serialdev,/dev/ttyACM0,9600n81,local
  enable: on
  options:
    banner: *banner
    kickolduser: true
    telnet-brk-on-sync: true
    rsize: 65536
    wsize: 65536
```

Then restart ser2net.

## Rebooting Pi kicks all active sessions

Expected. If you need sessions to survive a Pi reboot, use `screen`
with detach (Method 2 in [Access Methods](access-methods.md)) rather
than telnet — but even then, the Pi reboot terminates the underlying
serial connection.

## Multiple sessions to the same device

Default ConsolePi ser2net config sets `kickolduser: true` — new
sessions boot the old one. If you want multiple simultaneous read-only
observers (rare), you'd customize the ser2net stanza to disable
kickolduser and enable `remctl` or a similar multi-session mode.

For most people, `kickolduser: true` is the right behavior.

## Where to look for more

Ordered by usefulness:

1. Journal for the specific service: `sudo journalctl -u <service> --since '10 min ago'`
2. Install log (if still present): `/tmp/consolepi-install.log`
3. `/var/log/ConsolePi/*` — ConsolePi-specific logs
4. Upstream project: [github.com/Pack3tL0ss/ConsolePi/issues](https://github.com/Pack3tL0ss/ConsolePi/issues)
5. Discussion on [ArubaOS-CX Networkers Slack](https://slack.arubanetworks.com/) (ConsolePi's maintainer is active there)
