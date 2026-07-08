# Project-specific bindings for the shared rubrics

The general dispatch/judgment/delegation/maintenance rules now live in
`~/.claude/docs/` (shared across all projects on this machine). This file
holds ONLY the mama-app-specific concrete values that plug into those
general frameworks — the numbers, commands, and examples that used to live
inline in this project's own copies of those docs before 2026-07-08.

Read this alongside the shared docs, not instead of them:
`~/.claude/docs/dispatch.md`, `~/.claude/docs/judgment.md`,
`~/.claude/docs/delegation-templates.md`, `~/.claude/docs/maintenance.md`.

## Calibration for dispatch.md — do NOT spawn subagents by default here

The repo is one ~65KB file (`index.html`). Reading a section yourself (grep
banner → Read range, see CLAUDE.md) costs less than briefing a subagent to
do it. **Default here: do NOT spawn subagents for code reading or
single-file edits.** The main conversation does the work; agents are only
for: web research (browser API / Supabase behavior), independent
verification of risky work (see below), or parallel batch work if this
project ever grows past a few files.

Relevant skills: `/code-review` (diff review), `verify` (exercise a change
end-to-end), `run` (launch the app), `security-review` (anything
security-adjacent — read `docs/handoff.md` Part 2 first for the known
caveats).

This machine has **no node** — affects any syntax-check command you hand to
an agent; use the osascript pipeline below instead.

## "Risky" (judgment.md's verification trigger), concretely

A diff is risky here if it touches: the PIN gate, any data-deletion path,
accessibility CSS (font sizes, tap targets, colors), or is about to be
pushed. Reviewer model: `sonnet`; use `opus` when the diff touches
PIN/deletion/accessibility.

## Definition of done, concretely

- [ ] `git diff` re-read top to bottom; you can justify every hunk.
- [ ] Inline JS still parses. Run:
      ```
      awk '/^<script>$/{f=1;next} /^<\/script>$/{f=0} f' index.html > /tmp/app_check.js
      osascript -l JavaScript /tmp/app_check.js 2>&1 | head -2
      ```
      Reading the output: `SyntaxError` = you broke it, fix before anything
      else. `execution error` with TypeError/ReferenceError (about console/
      document/supabase) = **parse succeeded**; that's the expected PASS
      state, since there's no DOM outside a browser. (Verified working
      2026-07-07; this machine has no node.)
- [ ] No font-size decreased, no tap target shrunk, no genuinely new ad-hoc
      color. Check: `git diff | grep -iE "font-size|padding|width|height|min-width"`
      and compare old→new values by hand. Some controls are sized by
      width/height, not padding (`.mic` 76px, `.tgl` 52×28, `.wplay`) — eyeball
      those whenever the diff touches CSS.
- [ ] The new path fails gently in Chinese (visible message, e.g.
      `st.textContent='...請再試一次'`) — consistent with the section's
      pattern.
- [ ] The user confirmed the behavior matches what she approved (CLAUDE.md
      rule 2) — on her phone, if the change is user-visible.

PASS: after an accounting edit — syntax check run, diff shows only the
ACCOUNTING section changed, user replied「對，就是這樣」to a behavior summary.
FAIL: declaring done because the Edit tool call succeeded.
FAIL: "tests pass" — there are no tests here; that sentence is a
hallucination.

## Escalation example, concretely

PASS: Sonnet twice produced a broken edit to `playWord()`'s callback chain →
stop, escalate to opus with both failed diffs and the observed misbehavior
(this is the "requires reasoning across 3+ sections at once" trigger — the
`_pt` speech-cancellation scheme threads through playWord/playVocab/stopAll).
FAIL: escalating because a Chinese UI string "feels nuanced" — copy edits
never need escalation; ask the user instead.

## When to stop and ask the user, concretely

PASS: 「『記帳提醒改晚一點』— 改成 21:00 還是 21:30？首頁上顯示的『每天晚上 8:30』
文字要一起改嗎？」 (one message, both questions, concrete options)

Additive triggers on top of CLAUDE.md rule 2 (does not narrow it): decisions
about mom's preference (wording, reminder times, category names, colors,
layout); anything that deletes data or modifies a delete path; anything
touching the PIN, Supabase URL/key, or the `ai-proxy` request shape; before
every push, no standing authorization; whenever two readings of the request
would lead to different code.

## Wrong-direction example, concretely

PASS: second failed patch to video playback → revert, log lesson, re-read
the ALBUM section, notice the overlay resets `src` deliberately, take the
new seam.
FAIL: attempt #3 with more code piled on attempts #1–2 still in the file.

Also wrong-direction, specifically for this repo: raising the 10s/5MB video
caps, removing the AI call's timeout race, or skipping `esc()` on
interpolated strings — these mechanisms are load-bearing; find another way
or ask the user. If the AI proxy keeps failing, remember the app is
*designed* to degrade (local parsing, canned replies) — don't make it depend
harder on the proxy to "fix" that.

## Quality bar, concretely

- `var`/`function`, string concat; section placement under the right `════`
  banner (see CLAUDE.md / `docs/architecture.md`).
- All user/db strings interpolated into innerHTML pass through `esc()`.
- Failure paths visible in Chinese; success paths announced the way
  neighboring features do (e.g. `sayZh()` confirmations in ACCOUNTING).
- UI additions: primary/actionable text ≥19px; secondary/meta text may match
  the app's existing smaller patterns (13–18px, e.g. `.dt`, `.soft`) but
  never go below the closest existing analog. Tap targets comparable to
  existing buttons. Colors from the `:root` palette, or matching an existing
  hardcoded value used the same way (e.g. `CAT_COLOR` swatches) — never a
  genuinely new ad-hoc color.

## Delegation template fill-ins, concretely

When using `~/.claude/docs/delegation-templates.md` template 2 or 3
(Implementation / Refactor) for this repo, the constraints slot is always:
CLAUDE.md rules 1/4/5 apply verbatim — no font/tap-target reductions, style
must match exactly (`var`, `function`, string concat — `async`/`await` and
Promises are already used and fine, don't downgrade them, don't add new
`let`/`const`), no new files, Traditional-Chinese UI text, code placed under
the correct `════` banner section. The project's own syntax check is the
osascript pipeline above (there is no build step).

## Doc set this file is part of

This project's docs are `CLAUDE.md` + `docs/{architecture,handoff,lessons,
project-conventions}.md`, plus the shared `~/.claude/docs/{dispatch,
judgment,delegation-templates,maintenance}.md`. Maintenance protocol for
editing any of these: `~/.claude/docs/maintenance.md`.
