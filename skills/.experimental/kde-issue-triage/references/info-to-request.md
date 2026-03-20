# Reporter Info + Logs (Copy/Paste)

## Minimum environment info (always ask)
- OS + version (distro, kernel)
- Session: X11 or Wayland
- KDE Plasma version
- KDE Frameworks version (5.x or 6.x)
- Qt version (5.x or 6.x)
- Dolphin version (KDE Gear version)
- Repro frequency: always/sometimes; since when; last known good

Suggested commands (use what exists on the system):
```bash
dolphin --version
plasmashell --version
kwin_wayland --version || kwin_x11 --version
qmake --version || qtpaths6 --version || qtpaths --version
kf6-config --version || kf5-config --version
uname -a
```

## Clean profile checks (when config/state is suspected)
Ask the reporter to try a clean config (choose one approach, don’t spam options):

1) New user account (best signal).

2) Temporary config isolation (advanced; warn it changes behavior):
```bash
mkdir -p /tmp/kde-clean-config
XDG_CONFIG_HOME=/tmp/kde-clean-config XDG_DATA_HOME=/tmp/kde-clean-config dolphin
```

## Crash: backtrace requirements
- Ask for a backtrace with debug symbols (DrKonqi preferred on KDE).
- If using systemd-coredump:
```bash
coredumpctl --user list | head
coredumpctl --user info <PID_OR_COREDUMP_ID>
coredumpctl --user gdb <PID_OR_COREDUMP_ID>
```
In GDB:
- `bt full`
- `thread apply all bt full` (if it looks like deadlock/hang leading to crash)

## Hang/freeze: what to request
- Is the whole app frozen, or only the view (UI responds?)?
- CPU usage? (spinning vs deadlock)
- Stack traces if possible:
  - If user is comfortable with gdb/attach:
```bash
gdb -p $(pidof dolphin)
```
Then:
- `thread apply all bt full`

## KIO/file operation issues: logging
Ask them to run Dolphin from a terminal with KIO logging enabled and capture output:

KDE/Qt logging rules (pick a minimal set; expand only if needed):
```bash
export QT_LOGGING_RULES="*.debug=false;kf.kio.core.debug=true;kf.kio.gui.debug=true;kf.kio.widgets.debug=true"
dolphin
```

Also capture user journal around the time:
```bash
journalctl --user -b --no-pager | tail -n 300
```

## Filesystem / protocol specifics (high value)
- Local path or URL scheme: `file:/`, `trash:/`, `smb://`, `sftp://`, `mtp:/`, `recentlyused:/`
- If remote: auth method (ssh agent, key, password), server type (Samba/OpenSSH)
- If permissions: mount options, sandbox (Flatpak/Snap), portals

