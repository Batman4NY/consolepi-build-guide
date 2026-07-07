# Flash Raspberry Pi OS

Use the official [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
tool — not manual `dd`. Imager handles headless-setup config that would
otherwise require you to physically attach a monitor and keyboard to the Pi.

## Steps

1. **Download and install Raspberry Pi Imager** for your platform from
   raspberrypi.com/software.

2. **Insert your microSD card** into your workstation via a card reader.

3. **Launch Imager** and set:

   | Field | Value |
   |---|---|
   | Device | (leave alone or pick your Pi model) |
   | Operating System | **Raspberry Pi OS Lite (64-bit)** — "Other general purpose OS" → "Raspberry Pi OS (other)" → "Raspberry Pi OS Lite (64-bit)". |
   | Storage | Your microSD card |

4. **Click "Next", then "Edit Settings"** — this is the critical
   headless-configuration step:

   **General tab:**

   | Field | Value |
   |---|---|
   | Set hostname | `ConsolePi` |
   | Set username and password | `pi` / a password you'll remember |
   | Configure wireless LAN | **Leave OFF** — this box is wired-only. WiFi introduces failure modes we don't want. |
   | Set locale settings | Your timezone + keyboard layout |

   **Services tab:**

   | Field | Value |
   |---|---|
   | Enable SSH | **ON** — required for headless install |
   | Authentication | **Use password authentication** for first boot. You can add SSH keys after login. |

   **Options tab:**

   | Field | Value |
   |---|---|
   | Play sound when finished | Your call |
   | Eject media | ON |
   | Enable telemetry | Your call |

5. **Click "Save"** → **"Yes" to apply OS customization** → **"Yes" to
   erase and write**.

6. **Wait for write + verification** (~3-5 minutes). Imager will eject
   the card automatically when done.

## Verify before boot (optional but worthwhile)

Re-insert the SD card into your workstation and check that these files
exist in the `bootfs` partition:

- `firstrun.sh` (headless setup script)
- `userconf` (username/password hash — new format)
- `ssh` (empty file, indicates SSH enabled)

If the `firstrun.sh` file references your desired hostname, username,
and SSH-enabled state, you're good. Eject the card.

## First boot expectations

- Physically install the SD card in the Pi
- Connect Ethernet to your LAN switch
- Connect power
- Wait ~90 seconds for the Pi to run through first-boot config
- The Pi will reboot once automatically after applying `firstrun.sh`

## Finding the Pi on the network

After ~2 minutes total, the Pi should be reachable on your LAN as
`ConsolePi.local` (via mDNS/avahi):

```bash
ping ConsolePi.local
```

If that doesn't resolve, find its DHCP-assigned IP by:

- Checking your router's DHCP leases (look for hostname `ConsolePi` or
  MAC OUI starting with `b8:27:eb` or `dc:a6:32` or `d8:3a:dd`)
- ARP scanning: `arp-scan -l` or `nmap -sn 192.168.1.0/24`

## Next

Head to **[First Boot](first-boot.md)** for SSH setup and initial hardening.
