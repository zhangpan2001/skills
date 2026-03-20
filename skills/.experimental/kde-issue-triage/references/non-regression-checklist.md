# Non-regression Checklist (Manual Verification)

Use this to expand the "How You Can Verify" section. Pick the smallest set that meaningfully covers regression risk.

## General (almost always)
- Re-test the original repro steps (same data, same machine).
- Re-test once with a clean config (new user or `XDG_CONFIG_HOME` isolation) if config/state is involved.
- Check for obvious UX regressions: focus, selection, keyboard navigation, context menu actions.

## Dolphin view/model edge cases (KItemViews/KFileItemModel)
- Very large directory (10k+ items) for perf/regression signals.
- Mixed file types; sorting by name/date/size; toggling details/icons/compact views.
- Selection behaviors:
  - Shift range select, Ctrl toggle, click-drag selection box.
  - Rename (F2) and then selection changes.
- Hidden files toggle and filtering/search within the folder.

## File operations (copy/move/rename/delete)
- Rename collisions (existing filename) and undo/redo behavior if supported.
- Permission errors (read-only directory, no write permission).
- Long paths and non-ASCII filenames.
- Symlinks:
  - Operations on symlink vs target.
  - Broken symlink display/behavior.

## KIO / URL scheme coverage (when relevant)
- `file:/` local path.
- At least one special/virtual URL if related: `trash:/`, `recentlyused:/`.
- If remote/protocol-related, test the specific scheme involved:
  - `smb://` (Samba)
  - `sftp://` (OpenSSH)
  - `mtp:/` (phones)
- Cancellation: start the operation and cancel, ensure no stuck jobs/UI dead states.
- Network flakiness: transient disconnect/reconnect if the issue is remote.

## Qt/platform variables (only when symptoms match)
- Wayland vs X11 difference (if reproducible).
- DPI scaling:
  - `QT_SCALE_FACTOR=1` baseline if reporter used scaling.
- Theme/style influence: default style vs custom theme if UI glitchy.

## Crash/hang follow-ups (if applicable)
- If it used to crash: repeat the repro 3-5 times; confirm no new warnings.
- If it used to hang: verify job completes; verify no lingering threads/jobs; check `journalctl --user` for new critical logs.

