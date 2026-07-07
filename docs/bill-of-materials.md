# Bill of Materials

Total: ~$85 for the base build. Add ~$15 for a smart plug if you want
remote power cycle capability.

## Minimum build

| Item | Notes | Approx cost |
|---|---|---|
| **Raspberry Pi 3 Model B+** | Pi 3B+ is plenty for console-serving. Pi 4 and Pi 5 also work; Pi 5 is overkill unless you plan to run other services alongside. Pi Zero 2 W works but constrained. | $35 |
| **Official Pi PSU** (5V / 2.5A for Pi 3, 5V / 3A for Pi 4, 27W USB-C for Pi 5) | Don't cheap out on power. Undervoltage causes flaky USB behavior which will manifest as intermittent console drops. | $10 |
| **High-endurance microSD card, 32 GB** | SanDisk High Endurance or Samsung Pro Endurance recommended. Consumer cards die on the write-heavy workload of a long-running Linux system. | $12 |
| **Short Ethernet patch cable** | Cat5e or Cat6, however long you need to reach your rack switch. | $5 |
| **Console cable (choose based on your target)** | See "Console cable options" below. | $10-$15 |

## Console cable options

### For modern Cisco gear with USB-C console (1200/3100/4200 Secure Firewalls, Catalyst 9000, etc.)

- **Any USB-A to USB-C data cable** — a phone charging cable *if it supports
  data* (not all do; charge-only cables will not work). The Pi's USB-A end
  plugs into the Pi, USB-C plugs into the firewall/switch console port.
- **No FTDI or Prolific chip needed** — the target device itself acts as
  the USB-serial converter, presenting as `/dev/ttyACM0` on the Pi.

### For traditional RJ45 console

- **Cable Matters USB to Cisco Console Cable (FTDI variant)** — 6 ft,
  rollover pinout built in, uses FTDI FT232 chipset (excellent Linux
  driver support). Avoid the Prolific PL2303 variant — the FTDI works
  flawlessly, the Prolific has driver quirks under Trixie.
- Presents as `/dev/ttyUSB0` on the Pi.

### Both

If you'll be servicing both modern USB-C and legacy RJ45 gear, get both
cables. ConsolePi handles multiple adapters gracefully.

## Optional but recommended

| Item | Why | Approx cost |
|---|---|---|
| **Smart plug on the target device's power feed** | Enables remote power cycle. A Z-Wave, Zigbee, or Matter plug integrated with your existing home automation (Home Assistant, etc.) is ideal. TP-Link Kasa also works well as a standalone. | $15 |
| **Passive PoE splitter** | If your rack switch has PoE, powers the Pi from the same Ethernet cable — one cable to the Pi instead of Ethernet + PSU. Requires a Pi 4 or 5, or a passive PoE HAT for Pi 3. | $15 |
| **Small case with fan** | Pi 3B+ runs warm at load; a case with even a small fan extends SD card life and reduces thermal throttling. Argon NEO, official case, or a printed one. | $15 |
| **Second SD card** | For a spare / hot swap. Fleets grow. | $12 |

## What NOT to buy

- **Cheap generic "Cisco console cables" without a stated chipset** — they
  usually use Prolific clones with driver problems. Stick with named
  brands that clearly state FTDI.
- **USB hubs** unless you need to console more than 4 devices from a
  single Pi. Direct USB → target is more reliable.
- **Bluetooth serial adapters** — ConsolePi supports them, but they add
  failure modes with negligible benefit for a rack-mounted setup.

## Sourcing

- Raspberry Pi: authorized reseller (Adafruit, CanaKit, PiShop.us) — avoid
  Amazon third-party sellers, counterfeits are common
- Cables: Amazon, Micro Center, direct from Cable Matters
- Smart plugs: whatever integrates with your existing automation stack
