# First Boot

Get SSH working, apply updates, and lock down the basics before the
ConsolePi install.

## SSH in

From your workstation:

```bash
ssh pi@ConsolePi.local
# Accept the host key fingerprint (yes)
# Enter the password you set in Imager
```

If mDNS isn't resolving, use the IP from your router's DHCP lease list.

## Update the OS

Even a fresh Imager write is often a few weeks behind. Update immediately:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
```

If a kernel update was pulled, reboot:

```bash
sudo reboot
```

Wait ~30 seconds, SSH back in.

## Basic hardening (optional but recommended)

### Confirm WiFi is off

WiFi should already be off if you didn't configure it in Imager, but
verify:

```bash
nmcli radio wifi
# Should say: disabled
```

If it says `enabled`:

```bash
sudo nmcli radio wifi off
```

Persist across reboots by editing the WiFi connection profile (if any):

```bash
nmcli connection show
# For any wireless entries:
sudo nmcli connection modify <profile-name> connection.autoconnect no
```

### Add your SSH key (recommended)

If you'd rather not type the password every time:

```bash
# On your workstation:
ssh-copy-id pi@ConsolePi.local
```

Then optionally disable password auth in `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
```

Restart sshd: `sudo systemctl restart ssh`.

Test key auth in a NEW terminal (keep your existing session as a
fallback) before closing the password-auth session.

### Set a static IP (optional)

For rack use, a DHCP reservation on your router is usually easier than a
static IP on the Pi — the Pi stays configured to grab DHCP, and you
control the address at the router. The Pi's MAC starts with:

- `b8:27:eb` for Pi 3 and earlier
- `dc:a6:32` for Pi 4
- `d8:3a:dd` for Pi 5

If you insist on Pi-side static IP, use NetworkManager (Trixie ships
with it, not `dhcpcd` like older RaspiOS):

```bash
sudo nmcli connection modify "Wired connection 1" \
    ipv4.method manual \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns 192.168.1.1
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

Note that changing the IP over an SSH session will drop your SSH
connection — reconnect on the new IP.

## Verify everything looks sane before installing ConsolePi

```bash
# Pi identity
hostname
cat /proc/device-tree/model
uname -r

# OS
cat /etc/os-release | grep PRETTY
# Should say: PRETTY_NAME="Raspberry Pi OS 13 (trixie) Lite"
# or "Debian GNU/Linux 13 (trixie)"

# Network
ip -br addr
ip route

# Internet reach (ConsolePi installer needs GitHub)
curl -sSI https://github.com/ | head -1
# Should return: HTTP/2 200
```

If all four blocks look healthy, you're ready for the ConsolePi install.

## Next

Head to **[Install ConsolePi](install-consolepi.md)**.
