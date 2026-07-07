# Install ConsolePi

The ConsolePi installer is a one-line command that pulls the installer
script from GitHub and runs it. It supports both **interactive** and
**silent** modes.

For a lab / homelab setup with sensible defaults, **silent mode** is
much faster and easier to reproduce.

## The working silent-install command

Run this as the `pi` user (the installer will prompt for sudo internally):

```bash
wget -q https://raw.githubusercontent.com/Pack3tL0ss/ConsolePi/master/installer/install.sh -O /tmp/ConsolePi
sudo bash /tmp/ConsolePi \
    --silent \
    -p 'YourPassword' \
    --us \
    --tz America/New_York \
    -L \
    -6
```

Flag by flag:

| Flag | Meaning |
|---|---|
| `--silent` | No interactive prompts; use defaults + explicit flags for everything |
| `-p 'YourPassword'` | Password for the new `consolepi` user that gets created. Single-quote it. |
| `--us` | Set locale, WLAN regulatory domain, and keyboard to US. Use `--locale <cc>` if you want something else. |
| `--tz America/New_York` | Set the system timezone. Use your own tz. |
| `-L` | Auto-launch `consolepi-menu` on `consolepi` user login. Nice UX for SSH-in-and-pick-a-port. |
| `-6` | Disable IPv6. Simplifies things for a LAN-only device. Skip if you need IPv6. |

Full list: `sudo bash /tmp/ConsolePi --help` (after downloading it).

## What the installer does (~8 minutes on a Pi 3B+)

The install runs in three internal phases:

1. **Prep phase** — clones the ConsolePi repo, installs system deps
   (`ser2net`, `python3-*`, `nginx`, `openvpn`, `dnsmasq`, `hostapd`,
   `avahi-utils`, and about 50 more packages).
2. **Config phase** — normally interactive, but silent mode drives it
   from your flags.
3. **Deploy phase** — sets up the ConsolePi services, systemd units,
   ser2net config, udev rules, mDNS advertisement, cleanup timer.

Total time on a Pi 3B+ with a decent SD card: about 8 minutes. Longer
on a slower Pi or slower internet.

## What gets installed

Services (systemd):

- **`ser2net`** — presents attached serial adapters as telnet ports
  9000-9007 (`ttyACM0`-`ttyACM7`) and 8001-8008 (`ttyUSB0`-`ttyUSB7`)
- **`consolepi-api`** — REST API on `:5000` for programmatic control
- **`consolepi-mdnsreg`** — advertises this ConsolePi over mDNS
- **`consolepi-mdnsbrowse`** — discovers other ConsolePis on the LAN
- **`consolepi-cleanup`** — periodic housekeeping timer

User account:

- **`consolepi`** — new local user, member of `dialout`, `consolepi`,
  `sudo`. Password is whatever you set with `-p`. Home directory is
  `/home/consolepi/`. On SSH login, auto-launches `consolepi-menu` if
  you used `-L`.

Command shortcuts (available on `$PATH` for the `consolepi` user via
`/etc/profile.d/consolepi.sh`):

- `consolepi-menu` — interactive session picker
- `consolepi-addconsole` — add / rename serial adapter aliases
- `consolepi-details` — show attached adapters + their status
- `consolepi-showaliases` — list current adapter aliases
- `consolepi-help` — full command reference
- `consolepi-upgrade` — update the ConsolePi installation
- `consolepi-status` — service health
- 25+ others, see `consolepi-help`

## A watchable install (if you'd rather monitor)

Run the installer in a `screen` or `tmux` session so you can detach and
reattach:

```bash
sudo apt install -y screen
screen -S consolepi-install
# Then run the wget + install command inside screen
# Ctrl-A then D to detach; `screen -r consolepi-install` to reattach
```

Or just background it and tail the log:

```bash
nohup sudo bash /tmp/ConsolePi --silent -p 'YourPassword' --us \
    --tz America/New_York -L -6 > /tmp/consolepi-install.log 2>&1 &
tail -f /tmp/consolepi-install.log
```

## What "success" looks like

The installer prints a lot; watch for the final banner near the end:

```
Success Silent Install Complete a reboot is required.
```

You may see:

```
Warnings Occurred During Install (1).
```

The most common single warning is:

```
[WARNING][Bluetooth Console] rfcomm failed to start, may be normal
    depending on service/hardware
```

That's benign if you're not using Bluetooth serial. Ignore.

## Reboot

The installer says a reboot is required; take it:

```bash
sudo reboot
```

Wait ~40 seconds for it to come back, then SSH in again.

## Next

Head to **[Verify Install](verify-install.md)** to confirm everything
came up cleanly.
