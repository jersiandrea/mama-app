# Maintenance protocol for these docs

The docs are `CLAUDE.md` + `docs/{architecture,dispatch,judgment,
delegation-templates,maintenance,lessons,handoff}.md`. They are the project's
institutional memory; keep them true or they become poison.

## You may edit WITHOUT asking the user

- Append entries to `docs/lessons.md`. Required whenever a mistake cost more
  than ~10 minutes or produced a result the user had to correct.
- Fix factually stale references anywhere (renamed sections, dead paths, wrong
  tool/model names) — but verify the new fact (grep, run the command) before
  writing it. Fixing rot is duty, not permission-gated.
- Add PASS/FAIL examples to `docs/judgment.md` drawn from real events in your
  session.
- Update `docs/architecture.md` when the code has genuinely changed and the doc
  no longer matches (read the code first; the code is the truth).

## Ask the user BEFORE

- Changing, weakening, or deleting any Non-negotiable rule in `CLAUDE.md`.
- Changing the escalation ladder or the model/agent names in `docs/dispatch.md`
  (exception: marking a model as confirmed-unavailable after an Agent call
  rejected it — that's a stale-fact fix; note the date).
- Deleting or merging any doc file, or changing this protocol itself.
- Compacting `docs/lessons.md` (see below).

## Lesson entry format (append to docs/lessons.md)

```
## YYYY-MM-DD — <one-line title>
- What happened: <1–2 lines, concrete — name the file/function/command>
- Root cause: <1 line>
- Rule: <imperative and checkable next time, e.g. "grep for X before Y">
```

## Compaction

When `docs/lessons.md` exceeds ~15 entries or ~120 lines: propose a compaction
to the user — promote recurring lessons into `docs/judgment.md` rules, keep the
5 most recent raw entries, delete the rest. Never compact silently; never let
the file grow unbounded either.

## Size discipline

`CLAUDE.md` stays under ~60 lines. If a rule needs more than ~3 lines of
explanation, the explanation lives in `docs/` and `CLAUDE.md` keeps one line
plus a pointer. When adding to any doc, prefer replacing a weaker line over
appending a new one.
