# Lessons log

Append-only field notes. Format and compaction rules:
`~/.claude/docs/maintenance.md`.
Newest at the bottom.

## 2026-07-07 — No node on this machine
- What happened: planned to verify inline JS with `node --check`; `which node`
  came back empty.
- Root cause: assumed a dev toolchain exists; this is a plain macOS machine.
- Rule: before writing any shell command into docs or delegating it, run it
  once here first. The working JS check is the osascript pipeline in
  `docs/project-conventions.md`.

## 2026-07-07 — Fresh-eyes review caught 13 defects the author missed
- What happened: an independent review of the freshly written docs found the
  flagship grep command missing `-A1` (output had no section names), a false
  "ES5" label (the code uses async/await 31 times), and several rules that
  contradicted the actual codebase (root-only colors, ≥19px floor).
- Root cause: the author verified commands ran, but not that their OUTPUT
  matched what the doc promised; and described the code from memory of intent
  rather than re-checking it.
- Rule: reviewer must run each documented command and compare output against
  the doc's claim, literally, as a weak model would read it. Self-review does
  not count (`~/.claude/docs/dispatch.md` "never self-certify" — this entry
  is the proof).
