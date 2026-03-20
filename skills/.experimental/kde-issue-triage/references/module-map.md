# Symptom to Module Map (Heuristics)

Use this as a fast routing and "likely code path" helper. Keep it hypothesis-driven.

## Dolphin (app-level UI/workflow)
Common signals:
- Affects only Dolphin; other file dialogs or KIO clients behave differently.
- UI elements: panels, tabs, split view, selection, view mode, sorting, search/filter.

Likely areas:
- Dolphin UI shell and actions (main window, view container)
- View/model glue (selection, sorting, preview, status bar)
- KItemViews (item view behavior, selection quirks)

Ask: Does the same happen in the KDE file dialog? If not, bias toward Dolphin.

## KIO (jobs, protocols, URL handling)
Common signals:
- Copy/move/delete/rename failures, stalls, or wrong prompts.
- Remote protocols (`smb://`, `sftp://`, `mtp:/`) or special URLs (`trash:/`).
- Thumbnail/metadata operations hitting network unexpectedly.
- The same buggy behavior shows up through multiple KIO consumers, even if first reported in Dolphin.

Likely areas:
- KIO core job scheduling, error mapping, retry/prompt flows
- Protocol worker behavior (IO worker side) and auth handling
- KIO Widgets/FileWidgets integration (dialogs, confirmations)

Ask: Does it reproduce in another KIO client (e.g. file dialog, other KDE apps)?
If yes, treat Dolphin as the symptom surface and evaluate whether the proper fix belongs in KIO itself.

## KConfig (settings persistence and config corruption)
Common signals:
- Settings reset, not saved, inconsistent between sessions.
- A particular config file becomes corrupted; only happens after certain actions.
- Only affects state restoration / defaults.

Likely areas:
- INI parsing/escaping edge cases
- Group/key lookup and merge behavior
- Atomic writes / permissions / sandboxed config paths

Ask: Does it reproduce with a clean `XDG_CONFIG_HOME`?

## KDE Frameworks (shared libraries)
Common signals:
- Reproducible across multiple KDE apps.
- Behavior changes after Frameworks update without app update.

Likely areas:
- FileWidgets, KIO, KConfig, KWidgetsAddons, KItemViews
- Threading assumptions and event loop integration

Ask: Which Frameworks version changed? Is it a regression window?

## Qt (Widgets/QML/platform integration)
Common signals:
- Only on Wayland/X11, only with a particular style/theme, DPI scaling, IME.
- Rendering glitches, focus/keyboard navigation issues, QML bindings regressions.

Likely areas:
- Qt platform plugins, input/focus, high DPI scaling, item views, model roles

Ask: Does it reproduce with a default Qt style and scale factor 1?
