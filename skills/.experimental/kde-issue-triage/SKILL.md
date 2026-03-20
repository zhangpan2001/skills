---
name: "kde-issue-triage"
description: "Use when triaging and handling KDE community issues/bug reports for Dolphin, KIO, KConfig, KDE Frameworks, or Qt Widgets/QML; default to providing the smallest patch (no test changes) plus manual verification steps, and generate KDE-style commit messages only after the user confirms the fix."
---

# KDE Issue Triage (Dolphin/KIO/KConfig/KF/Qt)

## When to use
- KDE community issue handling: bug triage, reproduction, log/backtrace analysis, component routing, and proposing fixes.
- Deep dives for Dolphin, KIO workers, KConfig parsing/state, KDE Frameworks behavior, and Qt Widgets/QML UI issues.

## User preferences (default)
When the user requests a fix:
- Produce the **smallest reasonable patch** (minimal diff, minimal files).
- **Do not change tests** unless explicitly requested.
- After finishing source changes in the `dolphin` repo, run:
  - `kde-builder --no-include-dependencies dolphin`
- After finishing source changes in the `kio` repo, run:
  - `kde-builder --no-include-dependencies kio`
- Do **not** perform the final runtime bug verification yourself. The user will manually run:
  - `kde-builder --run dolphin`
- Instead, provide a short **manual verification checklist** for the user to run after the build succeeds.
- **Do not generate the final commit message yet**. Only generate commit message after the user confirms the bug is fixed and provides a valid `BUG: XXXX`.
- Optimize for **non-regression**: keep behavior changes tightly scoped; call out risks and edge cases; provide a verification checklist that covers likely regressions.
- When verification involves creating files, directories, symlinks, test data, or isolated config/data homes, provide **copy-paste shell commands**, not just prose steps.
- If the bug is caused by a function or behavior reached through the **KIO call chain**, explicitly evaluate whether the correct fix belongs in **KIO at the source** rather than only in Dolphin/the caller, and tell the user that assessment.

## Output expectations (always)
- Be professional, clear, patient, and actionable.
- Prefer evidence-driven hypotheses over speculation; explicitly call out assumptions.
- If info is missing, ask for it with copy/paste commands.
- If proposing a patch, include: likely files/classes, minimal change strategy, tests, and risk.
- If asked for a commit message, follow the strict template in `references/commit-message.md`.
  - Use KDE-style commit formatting: subject line, blank line, body paragraphs, blank line, `BUG: XXXX`.
  - Prefer the scope of the core file/class actually changed, especially when the patch only touches one file.

## Default response structure
1. **Summary**
   - One paragraph: what the bug seems to be; impact; whether it smells like regression.
2. **Triage**
   - Severity: crash/hang/data loss/security/perf/UI/cosmetic.
   - Repro: always/sometimes; clean profile; new user; specific filesystem/protocol.
   - Scope: Dolphin-only vs KIO-wide vs Frameworks.
3. **What I Need From You**
   - Ask only for missing essentials; provide exact commands.
   - Use `references/info-to-request.md` for standard info/log collection.
4. **Likely Code Path**
   - Point to modules + 2-3 candidate classes/files; explain why.
   - Use `references/module-map.md` when mapping symptoms -> component.
5. **Root Cause Hypotheses**
   - 2-3 hypotheses max; each with supporting signals and how to falsify.
6. **Fix Direction**
   - Minimal fix first; mention behavioral risks; mention relevant tests (planning only).
   - If the failure originates from KIO-shared behavior, explicitly say whether the fix should move upstream into KIO.
7. **Minimal Patch**
   - Provide a unified diff implementing the smallest plausible fix.
8. **How You Can Verify**
   - Provide a short, concrete checklist (exact UI steps / commands) for the user to validate whether the bug still exists.
   - If setup requires files/directories/test data, include exact shell commands the user can copy directly.
   - For Dolphin/KIO source changes, mention the expected build command first; for Dolphin fixes, follow it with the user's manual `kde-builder --run dolphin` step.
9. **Next Actions**
   - For reporter: what to try next.
   - For maintainer: how to proceed (labels, duplicates, regression tagging, MR plan).
   - If the user confirms the fix and provides `BUG: XXXX`, generate commit message from `references/commit-message.md`.
   - When generating the title, bias toward the core file/class that contains the real fix logic.

## Triage workflow (procedural)
### 1) Normalize the report
- Rewrite the issue into: Expected / Actual / Steps / Frequency / Environment.
- Identify if it is:
  - Downstream packaging issue (distro patch, sandbox, portals).
  - Upstream KDE issue (reproducible with upstream builds).
  - External dependency (GVFS, Samba, ssh-agent, kernel fs, Qt bug).

### 2) Confirm environment and versions
- Always request exact versions for Dolphin, KDE Frameworks, Qt, Plasma, and the OS.
- If the report mentions "latest" or "after update", anchor to concrete version numbers and dates.
- If KDE 5 vs 6 is ambiguous, ask: "Is this Plasma 5 or Plasma 6? Frameworks 5 or 6?"

### 3) Decide the owning component
- If the symptom is file operations, remote URLs, thumbnails, dialogs, or protocols, bias toward KIO/FileWidgets.
- If it is settings persistence, config corruption, or state reset, bias toward KConfig/KConfigCore.
- If it is UI navigation/panels/tabs/selection/search in Dolphin, bias toward Dolphin + KItemViews.
- If Dolphin only exposes the symptom but the faulty logic is shared through KIO, prefer fixing it at the KIO source and say so explicitly.
- Use `references/module-map.md` to quickly map the symptom to likely code owners.

### 4) Reproduction strategy
- Ask for:
  - Can reproduce with a new user / clean config?
  - Can reproduce with `--new-window` and no plugins/addons?
  - Does it reproduce on local filesystem vs `smb://`, `sftp://`, `trash:/`, `mtp:/`?
- If crash: require a backtrace with debug symbols; prefer DrKonqi or `coredumpctl gdb`.
- If hang: request stack traces (thread dumps) and clear "what is frozen" details.

### 5) Logging strategy (pick the minimal set)
- If it's KIO-related: turn on KIO logging rules; capture terminal output + `journalctl --user`.
- If it's Qt model/view weirdness: ask for a minimal repro and check if it persists with `QT_SCALE_FACTOR=1` and without third-party Qt styles.
- Use the commands in `references/info-to-request.md`.

### 6) Proposing a fix
- Prefer small, reviewable patches.
- Include a test plan (planning only; do not modify tests unless asked):
  - Unit tests if logic is in Frameworks (e.g., KConfig parsing, URL handling).
  - Integration tests if behavior is KIO job scheduling or filewidget flows.
  - Manual steps if UI; be explicit and short.
- If the patch is in the `dolphin` repository, compile with `kde-builder --no-include-dependencies dolphin` after editing.
- If the patch is in the `kio` repository, compile with `kde-builder --no-include-dependencies kio` after editing.
- Provide MR guidance: targeted change, link to issue, and commit message template.

## Patch requirements (when user wants a patch only)
- Keep diffs minimal: avoid refactors, reformatting, renames, or unrelated cleanups.
- Touch the fewest files possible.
- If multiple fixes are possible, pick the lowest-risk minimal change and explain why.
- If the current bug is downstream of a KIO-shared function, compare:
  - local caller-side patch
  - KIO source-level patch
- Prefer the KIO source-level fix when it is the real ownership boundary, and explain that recommendation clearly to the user.
- Always include:
  - The patch (unified diff).
  - A brief rationale (1-3 sentences).
  - A verification checklist the user can run (include likely regression scenarios).
  - For Dolphin/KIO fixes, note the exact post-edit build command that should be run before manual verification.
  - Copy-paste shell commands for any verification setup that can be automated:
    - creating/removing files or directories
    - creating symlinks
    - preparing sample data or long paths
    - launching with temporary `XDG_CONFIG_HOME` / `XDG_DATA_HOME`
  - A short **Non-regression notes** section:
    - What existing behavior might change (even subtly) and why it should not.
    - What edge cases could be affected and how to verify them.
    - Any invariants that must remain true (e.g., model/view roles consistency, signal ordering, thread affinity).
  - A note that commit message comes after confirmation + `BUG: XXXX`.

## Community etiquette and issue hygiene
- Be kind; assume good intent.
- Don’t block on perfect reports: ask for missing info, but also provide immediate troubleshooting.
- Handle duplicates:
  - If clearly duplicate: link canonical issue, explain why, close as duplicate.
  - If partially overlapping: keep open but cross-link.
- Regression handling:
  - Ask for "last known good" version.
  - If reporter can bisect, provide high-level instructions; otherwise propose suspect areas.

## MR and testing guidance (high-level)
- Prefer building with `kde-builder` / `kdesrc-build` workflows used by KDE contributors.
- For CMake + Ninja based builds:
  - Keep incremental builds; run the smallest relevant test subset when possible.
  - If no tests exist for the area, note the gap, but do not add tests unless the user asks.
- When asked for a commit message, load `references/commit-message.md` and follow it exactly.

## References to load on demand
- `references/info-to-request.md`: commands and minimal info to request from reporters.
- `references/module-map.md`: symptom -> module/class hints for Dolphin/KIO/KConfig/KF/Qt.
- `references/commit-message.md`: strict commit message template required by the user.
- `references/non-regression-checklist.md`: common KDE/Dolphin/KIO edge cases to include in manual verification.
- `references/verification-command-conventions.md`: how to write copy-pasteable verification setup commands.
