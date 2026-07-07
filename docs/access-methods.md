# Access Methods

There are three ways to reach a device connected to the ConsolePi.
Pick whichever is more convenient for the moment — they can coexist.

## Method 1: Telnet to the ser2net port (fastest)

Direct pass-through — no login to the ConsolePi itself, just straight
to the serial console.

```bash
telnet ConsolePi.local 9000     # ttyACM0 (Cisco USB-C, etc.)
telnet ConsolePi.local 8001     # ttyUSB0 (FTDI/PL2303, etc.)
```

Or by IP:

```bash
telnet 192.168.1.142 9000
```

Exit: `Ctrl-]` then `quit`.

**When to use this:** you know exactly which port maps to which device,
and you just want a session immediately.

**Watch out for:** telnet is cleartext. Fine for a lab or trusted LAN.
For anything over an untrusted network, tunnel it through SSH (below).

### Kicked out unexpectedly?

By default, ser2net's `kickolduser: true` setting means a **newer telnet
session boots the older one**. If a colleague or a stale terminal is
holding the port, your new session takes it and theirs drops. Feature,
not a bug.

## Method 2: SSH + `screen /dev/tty…` (bypass ser2net)

For when you want SSH-encrypted access, or ser2net is misbehaving, or
you want to attach with a different baud rate.

```bash
ssh pi@ConsolePi.local
sudo screen /dev/ttyACM0 9600
```

`screen` command reference:

| Keystroke | Action |
|---|---|
| `Ctrl-A` then `d` | Detach (leave screen running in background) |
| `Ctrl-A` then `k` then `y` | Kill the screen session entirely |
| `Ctrl-A` then `?` | Show all keybindings |
| `Ctrl-A` then `[` | Enter scrollback mode (arrows to scroll, `q` to exit) |

Re-attach a detached session:

```bash
sudo screen -r
```

**When to use this:** you want SSH-level security, or you need to
temporarily change baud rate, or you want the session to survive your
SSH disconnect.

## Method 3: `consolepi-menu` (interactive picker)

The friendly UI for when you have multiple adapters and don't want to
remember which port maps to which device.

```bash
ssh consolepi@ConsolePi.local
# If installed with -L flag, consolepi-menu launches automatically
# Otherwise: type `consolepi-menu`
```

The menu shows:

- Locally attached serial adapters (with alias names if you've set them)
- Remotely attached adapters on other ConsolePis discovered via mDNS
- Numeric picker — type a number, press Enter, land in the session

Exit the menu to shell: `x` then Enter.

Exit a serial session back to the menu: `Ctrl-A` then `d` (screen-style
detach), or the ConsolePi-specific menu commands.

!!! warning "consolepi-menu needs a real TTY"
    Running `consolepi-menu` in a non-interactive context (SSH `-c`
    command, script, cron) fails with:
    ```
    TypeError: unsupported operand type(s) for -: 'NoneType' and 'int'
    ```
    at `tty.rows - sum(...)`. This is because there's no PTY assigned
    and terminal size can't be computed. Interactive SSH sessions work
    fine. Details in **[Trixie Gotchas](trixie-gotchas.md#consolepi-menu-crashes-in-non-interactive-shells)**.

## Which to use, when

| Situation | Best method |
|---|---|
| One device, quick session | Telnet (Method 1) |
| Multiple devices, don't remember port mapping | consolepi-menu (Method 3) |
| Need SSH encryption end-to-end | Method 2 |
| Non-standard baud rate | Method 2 |
| Automating (Ansible, expect scripts) | Telnet direct (Method 1) |
| Session must survive disconnect | Method 2 with detach |

## Alias adapters for stable naming

If you have multiple adapters and want stable, human-readable names
(instead of `/dev/ttyUSB0` shifting between reboots):

```bash
sudo consolepi-addconsole
```

Follow the interactive prompts. It uses the adapter's udev-populated
serial number to create a persistent alias that survives reboots and
plug order changes. Aliases appear in `consolepi-menu` and can be used
directly:

```bash
telnet ConsolePi.local <alias-port>
```

Existing aliases:

```bash
consolepi-showaliases
```

## Multiple ConsolePis (fleet)

If you build more than one ConsolePi, they auto-discover each other
via mDNS (`consolepi-mdnsbrowse` service). `consolepi-menu` on any of
them shows all locally-attached AND remotely-attached adapters across
the fleet — you can pick a device on another ConsolePi and it
transparently proxies the session.

For a rack with multiple sub-rack zones or multiple sites, this scales
nicely without a central config.

## Next

If any of these paths aren't working, head to
**[Troubleshooting](troubleshooting.md)**.
