# Commit Message Template (KDE Style)

Follow the spirit of https://cbea.ms/git-commit/#seven-rules, but format the final message in KDE style.

Format:

```text
where: short imperative summary

Why / impact paragraph in normal prose.

How / approach paragraph in normal prose.

BUG: XXXX
```

Rules:
- Do **not** include helper labels like `We want our commits...`, `[Why]`, `[How]`, or `[BUG: XXXX]`.
- The first line should be a normal subject, followed by a blank line.
- Prefer the **narrowest accurate scope** for `where:` based on the code actually changed:
  - Good: `kfileitemmodel:`, `kitemlistview:`, `kio:`, `kfileitemmodelrolesupdater:`
  - Too broad unless truly appropriate: `dolphin:`
- The title should land on the **real modification scope** and, when possible, the **core file/class that fixes the bug**.
- If the patch mainly changes **one file**, use that file's main responsibility/class as the scope by default.
- Only fall back to broader scopes like `dolphin:` or `kio:` when the fix truly spans multiple files/modules or the change is genuinely module-wide.
- Prefer the file/class that contains the **actual bug fix logic**, not just the caller or symptom surface.
- The subject should describe the user-visible or code-level change succinctly, in imperative style.
- The body should be plain paragraphs, not labeled sections.
- Keep body text focused on:
  - why the old behavior was wrong
  - what the patch changes
- Do not mention manual verification in the commit message unless the user explicitly asks for that.
- End with:
  - `BUG: XXXX`
  - no brackets
- If no bug ID is available, ask the user for it before generating the final commit message.

Example:

```text
kfileitemmodel: sort dotted numeric names naturally

Natural sorting handled plain numeric chunks, but names containing dots between numeric segments were still ordered lexically in important cases. That broke expected ordering for decimal-style names like 0.09 and 0.1, and also for version-like names such as v1.2.3 and v1.2.10.

Teach KFileItemModel's natural string comparison to recognize dotted numeric chains instead of relying only on QCollator's numeric mode. Compare two-part numeric chains as decimal values, compare longer chains segment by segment like version numbers, and still split real file extensions separately so names like 1.09.txt keep working correctly.

BUG: 411707
```

Scope selection order:
1. Core file/class actually changed to fix the bug
2. Shared helper/model/view class if that is the true logic owner
3. Broader subsystem only if the patch is genuinely cross-cutting
