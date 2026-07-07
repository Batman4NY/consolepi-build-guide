# Verify Install

After the post-install reboot, run through these checks. Total time: 2
minutes.

## Services

All five ConsolePi services should be `active`:

```bash
for svc in ssh ser2net consolepi-api consolepi-mdnsreg consolepi-mdnsbrowse; do
    printf '  %-25s %s\n' "$svc" "$(systemctl is-active $svc)"
done
```

Expected output:

```
  ssh                       active
  ser2net                   active
  consolepi-api             active
  consolepi-mdnsreg         active
  consolepi-mdnsbrowse      active
```

If anything is `inactive` or `failed`, jump to
**[Troubleshooting](troubleshooting.md)**.

## Listening ports

```bash
sudo ss -tlnp | grep -E 'ser2net|consolepi-api|sshd' | sort -k4
```

Expected — port `22` (SSH), `5000` (API), `8001-8008` (`ttyUSB*` if
adapters attached), `9000-9007` (`ttyACM*` if adapters attached).

`ser2net` opens the tty ports even when no adapter is attached — they
just don't have anything to talk to yet.

## mDNS advertising

From your workstation (not the Pi):

```bash
avahi-resolve -n ConsolePi.local
# or
getent hosts ConsolePi.local
```

Expected: returns the Pi's LAN IP.

!!! warning "mDNS advertised IP quirk"
    On Trixie, `consolepi-mdnsreg` sometimes advertises the internal
    hotspot IP (`10.110.0.1`) in the A record instead of the real
    eth0 IP. The real IP is still in the TXT record's `rem_ip` field.
    Details: **[Trixie Gotchas](trixie-gotchas.md#mdns-a-record-shows-the-hotspot-ip)**.

## consolepi-details

Log in as the `consolepi` user (or use `sudo -u consolepi`) and run:

```bash
consolepi-details
```

This shows attached serial adapters, their ser2net ports, and mDNS
state. If you haven't plugged anything in yet, adapters will be empty.

## Attach a serial adapter and see it detected

Plug a USB console cable (FTDI or USB-C-to-Cisco) into any Pi USB port.

Check enumeration:

```bash
lsusb | grep -iE 'cisco|ftdi|prolific|silabs|serial'
ls /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
```

You should see something like:

```
Bus 001 Device 005: ID 05a6:0009 Cisco Systems, Inc. Console
/dev/ttyACM0
```

(For an FTDI cable it'd be a different VID/PID and `/dev/ttyUSB0`.)

## First console session

If the adapter is at `/dev/ttyACM0`, telnet to the ser2net port:

```bash
telnet ConsolePi.local 9000
```

You should see the ser2net banner and then whatever the connected
device's console shows. Hit `Enter` to elicit a prompt.

Exit: `Ctrl-]` then `quit`.

## Health check summary

If all of these are green, you're done — the ConsolePi is fully
operational:

- [x] All 5 services `active`
- [x] Port 22 + 5000 + 9000-9007 or 8001-8008 listening
- [x] `ConsolePi.local` resolves
- [x] `consolepi-details` runs (as `consolepi` user)
- [x] Plugged-in adapter appears in `lsusb` + `ls /dev/tty*`
- [x] Telnet to the ser2net port shows the target's console

## Next

Head to the connection guides:

- **[Connect via USB-C](connect-cisco-usb-c.md)** for modern Cisco gear
- **[Connect via RJ45 (FTDI)](connect-cisco-rj45.md)** for traditional path
- **[Access Methods](access-methods.md)** to understand the different
  ways to reach an attached device
