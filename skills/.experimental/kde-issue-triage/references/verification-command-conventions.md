# Verification Command Conventions

Use these rules when writing the "How You Can Verify" section.

## Core rule
- If manual verification requires preparing files, directories, symlinks, sample data, or temporary config/data homes, provide **copy-paste shell commands**.
- Do not describe setup only in prose when a command can do it.
- For source changes in KDE repositories, include the exact build command before the manual run command.

## Output format
- Prefer short fenced `bash` blocks.
- Keep commands directly runnable.
- Use stable temporary paths under `/tmp/` unless the user provided a project-specific path.
- If cleanup is useful, provide a separate cleanup command block.

## Good patterns

### Create a clean temp directory
```bash
rm -rf /tmp/dolphin-bug-repro
mkdir -p /tmp/dolphin-bug-repro
```

### Create sample files
```bash
touch /tmp/dolphin-bug-repro/a.txt
touch /tmp/dolphin-bug-repro/b.txt
mkdir -p /tmp/dolphin-bug-repro/subdir
```

### Create files with spaces / non-ASCII / long names
```bash
touch "/tmp/dolphin-bug-repro/file with spaces.txt"
touch "/tmp/dolphin-bug-repro/测试文件.txt"
python3 - <<'PY'
from pathlib import Path
base = Path("/tmp/dolphin-bug-repro")
name = "a" * 120 + ".txt"
(base / name).touch()
PY
```

### Create symlinks
```bash
ln -s /tmp/dolphin-bug-repro/a.txt /tmp/dolphin-bug-repro/a-link.txt
ln -s /tmp/dolphin-bug-repro/missing-target /tmp/dolphin-bug-repro/broken-link.txt
```

### Run with isolated config/data homes
```bash
rm -rf /tmp/dolphin-clean-config /tmp/dolphin-clean-data
mkdir -p /tmp/dolphin-clean-config /tmp/dolphin-clean-data
XDG_CONFIG_HOME=/tmp/dolphin-clean-config XDG_DATA_HOME=/tmp/dolphin-clean-data dolphin /tmp/dolphin-bug-repro
```

### Cleanup
```bash
rm -rf /tmp/dolphin-bug-repro /tmp/dolphin-clean-config /tmp/dolphin-clean-data
```

## KDE-specific guidance
- For Dolphin/KIO bugs, prefer commands that create a deterministic local repro under `/tmp/`.
- If the issue is about selection/sorting/renaming, create the exact files and folders needed for the scenario.
- If the issue is about config/state, always prefer an explicit `XDG_CONFIG_HOME` / `XDG_DATA_HOME` command over prose.
- If the bug fix is in the `dolphin` repo, include:
```bash
kde-builder --no-include-dependencies dolphin
```
- If the bug fix is in the `kio` repo, include:
```bash
kde-builder --no-include-dependencies kio
```
- After the build command, tell the user to do the runtime check manually with:
```bash
kde-builder --run dolphin
```
- If the issue needs multiple verification scenarios, separate them into:
  - Setup commands
  - Launch command
  - UI steps
  - Optional cleanup command
