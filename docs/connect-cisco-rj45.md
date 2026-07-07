# Connect via RJ45 (traditional FTDI path)

For gear without USB-C console (most Cisco equipment from before ~2022),
the traditional path is an **RJ45 console port + USB-to-serial adapter
cable**.

## What you need

- **Cable Matters USB to Cisco Console Cable — FTDI variant**, 6 ft
  (or equivalent from Tripp Lite, StarTech, etc.)
- Must specify **FTDI FT232 chip** (not Prolific PL2303 — the Prolific
  variant has driver quirks under Trixie)
- The cable has an RJ45 connector on one end with rollover pinout built in

## Physical connection

- USB-A end into any USB port on the Pi
- RJ45 end into the target's console port (usually blue, labeled CONSOLE)

## What the Pi sees

The FTDI cable enumerates as a USB-serial adapter:

```bash
lsusb | grep -iE 'ftdi|future technology'
# Bus 001 Device 006: ID 0403:6001 Future Technology Devices International, Ltd FT232 Serial (UART) IC
```

And a `/dev/ttyUSB*` character device appears:

```bash
ls /dev/ttyUSB*
# /dev/ttyUSB0
```

Udev populates useful attributes:

```bash
udevadm info -q property /dev/ttyUSB0 | grep -E '^ID_'
# ID_VENDOR=FTDI
# ID_MODEL=FT232R_USB_UART
# ID_MODEL_ID=6001
# ID_SERIAL=FTDI_FT232R_USB_UART_ABC123XY
# ID_VENDOR_ID=0403
```

## ser2net port mapping

ConsolePi's ser2net config maps `/dev/ttyUSB*` devices to telnet ports:

| Device | Telnet port |
|---|---|
| `/dev/ttyUSB0` | 8001 |
| `/dev/ttyUSB1` | 8002 |
| `/dev/ttyUSB2` | 8003 |
| `/dev/ttyUSB3` | 8004 |
| `/dev/ttyUSB4` | 8005 |
| `/dev/ttyUSB5` | 8006 |
| `/dev/ttyUSB6` | 8007 |
| `/dev/ttyUSB7` | 8008 |

## Talk to the device

From any host on your LAN:

```bash
telnet ConsolePi.local 8001
```

Hit `Enter` to elicit a prompt. Default baud rate is 9600, which is
correct for virtually all Cisco console ports.

Exit telnet: `Ctrl-]` then `quit`.

## Alternate: SSH + `screen`

```bash
ssh pi@ConsolePi.local
sudo screen /dev/ttyUSB0 9600
```

Exit `screen`: `Ctrl-A` then `k` (kill) or `Ctrl-A` then `d` (detach).

## Baud rates for other vendors

Not every vendor uses 9600 for console. If you're consoling a
non-Cisco device:

| Vendor | Common console baud |
|---|---|
| Cisco (most) | 9600 |
| Cisco UCS C-series server (BIOS/CIMC redirect) | 115200 |
| Juniper | 9600 |
| Arista | 9600 |
| Palo Alto | 9600 |
| Fortinet | 9600 |
| Dell PowerEdge iDRAC serial | 115200 |
| HPE ProLiant iLO serial | 9600 or 115200 (check iLO config) |

To use a different baud rate at connect time via ser2net, you'd need
to add a custom port stanza in `/etc/ser2net.yaml`, or connect via
`screen /dev/ttyUSB0 115200` directly.

## Multiple devices

Same as USB-C — 4 direct USB ports on the Pi, more via powered USB hub.
The `/dev/ttyUSB*` device numbers are assigned in USB enumeration order,
which can shift between reboots. For stable naming, use the udev-populated
`/dev/serial/by-id/` path or use `consolepi-addconsole` to create
persistent aliases.

## Watch out for

- **Rollover cable + straight-through RJ45** doesn't work. Cisco console
  needs rollover pinout, which is built into console cables specifically.
  A random RJ45 patch cable will not work in place of a console cable.
- **RJ45 console and USB console** — on newer Cisco gear that has both,
  USB wins if both are plugged in. Unplug the USB-C side if you want
  to force the RJ45 console.
- **Prolific PL2303 cables** are cheaper but have Linux driver quirks
  that manifest as intermittent character drops. Stick with FTDI.

## Next

- **[Access Methods](access-methods.md)** — telnet, SSH, `consolepi-menu`
