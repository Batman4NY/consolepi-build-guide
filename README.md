# ConsolePi Build Guide

A tested, step-by-step recipe for building a Raspberry Pi into a full-featured
serial console server for Cisco (and other) network gear — using
[ConsolePi](https://github.com/Pack3tL0ss/ConsolePi) on Raspberry Pi OS 13
(Trixie).

Live web version: **https://batman4ny.github.io/consolepi-build-guide/**

## Why this exists

The upstream ConsolePi project is excellent, but two things aren't well-covered
elsewhere:

1. **What actually works on Raspberry Pi OS 13 (Trixie)** — most walkthroughs
   target Bullseye or Bookworm; Trixie brings NetworkManager, Python 3.13, and
   ser2net v4 changes that shift the failure modes.
2. **How to connect it to modern Cisco USB-C consoles** — the 1200/3100/4200
   Secure Firewall series and Cisco Catalyst 9000 switches expose USB-C
   console ports that enumerate as USB-CDC devices (`/dev/ttyACM*`), which
   many console-server recipes still don't cover. No FTDI cable needed.

This guide is written from a working, verified build on a Raspberry Pi 3B+
with Trixie, connected to a Cisco Secure Firewall 1210CE via USB-A → USB-C.

## What you'll build

A network-accessible serial console server that:

- Presents attached serial devices as telnet ports (one per adapter)
- Auto-advertises itself via mDNS (`consolepi.local`)
- Runs on quiet, low-power hardware you can leave in a rack
- Supports both traditional RJ45 console cables (FTDI USB-serial) and
  modern USB-C console ports (native CDC-ACM)
- Can be paired with a smart plug for full out-of-band (console + power-cycle)

Estimated cost: ~$85 in parts (Pi + PSU + cable + SD).
Estimated build time: 45 minutes from unboxing to first console session.

## Guide structure

1. **[Overview](docs/index.md)** — what this is, who it's for, what it isn't
2. **[Bill of materials](docs/bill-of-materials.md)** — exact parts, with alternates
3. **[Flash Raspberry Pi OS](docs/flash-raspios.md)** — Imager settings
4. **[First boot](docs/first-boot.md)** — SSH in, initial hardening
5. **[Install ConsolePi](docs/install-consolepi.md)** — the working silent-install recipe
6. **[Verify install](docs/verify-install.md)** — what "healthy" looks like
7. **[Connect via USB-C](docs/connect-cisco-usb-c.md)** — the modern path
8. **[Connect via RJ45 (FTDI)](docs/connect-cisco-rj45.md)** — the traditional path
9. **[Access methods](docs/access-methods.md)** — telnet, SSH, `consolepi-menu`
10. **[Trixie gotchas](docs/trixie-gotchas.md)** — Debian 13 / RaspiOS 13 specifics
11. **[Troubleshooting](docs/troubleshooting.md)** — symptoms, causes, fixes

## Contributing

Corrections, additional platform notes, and troubleshooting entries are
welcome. Open an issue or PR. This guide is intentionally opinionated — it
documents what actually works, not every possible configuration.

## License

Apache-2.0. See [LICENSE](LICENSE).
