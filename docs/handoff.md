# Handoff — written 2026-07-07 by a Fable 5 session

The user asked that session to encode its judgment into files so weaker models
(Sonnet/Opus/Haiku) operate well here. This file holds: the harness diagnosis that
the other docs are built on, a letter to future sessions, a degradation forecast,
and an honest statement of what this system cannot fix.

## Part 1 — Diagnosis: the three biggest problems in this harness

### 1. Biggest token waste: re-reading all of index.html
The entire app is one ~1,125-line / ~65KB file. A naive session reads it whole
(measured 2026-07-07: the first 840 lines alone cost ~28k tokens; the full file
is roughly 35–40k), then re-reads it after edits. Multiply by every session.

**Fix (encoded in CLAUDE.md → "Working in index.html"):** never read the whole
file. Locate the section first with `grep -n -A1 "════" index.html` (the `-A1`
matters — the section name is on the line after the border; banner comments
divide the JS into 11 named sections), then Read only that range. Never re-read
a file after your own successful Edit — the tool errors if an edit fails.

### 2. Easiest place to cause real damage: push-to-main is production
`git push` to `main` deploys instantly to https://jersiandrea.github.io/mama-app
— no CI, no tests, no staging — and the live user is the developer's mother, who
has poor eyesight and cannot file a bug report. A change that "looks fine" in a
diff can break her phone (the code leans on `webkitSpeechRecognition` and
`-apple-system` fonts — likely iPhone Safari, though the exact device is
UNVERIFIED; ask the user).

**Fix (encoded in CLAUDE.md rule 3 + docs/project-conventions.md's "Definition
of done, concretely"):** commits are allowed when asked; pushes only on
explicit user instruction, and only after that checklist passes — including
the `osascript` JS syntax check (this machine has **no node**; the osascript
pipeline was tested and works, see project-conventions.md for the exact
command and how to read its output).

### 3. Easiest way to lose the plot: over-engineering by default
Weaker models reflexively "improve" things: add a package.json, split files,
introduce a framework, modernize `var` to `const`, tighten spacing to look
cleaner. Every one of those is wrong here. The single-file ES5 design is
deliberate (owner maintainability), and "cleaner/denser UI" directly harms the
low-vision primary user.

**Fix (encoded as CLAUDE.md rules 1 and 4):** hard bans, not guidelines — no new
tooling, no file splits, no font/tap-target/contrast reductions ever. Plus
`~/.claude/docs/judgment.md` §4 lists these as wrong-direction signals: if
your plan involves one, the plan is wrong by definition, don't retry harder.

## Part 2 — Letter: three things the user didn't ask, but you must know

### 1. The data is effectively public — do not let this surprise the family
`index.html` is public on GitHub and GitHub Pages, and it contains the PIN and
the Supabase URL + publishable key in plain text. The PIN gates nothing but the
UI. Whether Supabase Row Level Security is enabled is UNVERIFIED from this repo
— if RLS is off, anyone on the internet can read and delete the family's
expenses, photos, and videos directly via the Supabase API, no PIN needed.
Do not "fix" this unilaterally (you could lock mom out). But the first time the
user asks for anything security-adjacent, or reports strange data, raise this
first and offer to check RLS together in the Supabase dashboard.

### 2. The database will silently fill up
Photos (~900KB base64) and videos (up to ~6.7MB as base64) are stored inline in
`album_posts` rows — no Storage bucket, no CDN. Supabase free tier is 500MB.
Expired posts are deleted only opportunistically when someone opens the album
tab, and `accounts` grows forever (the in-app cleanup is manual). If the user
reports "分享失敗" or general slowness, check database size before debugging code.

### 3. The verification bottleneck is a human, and it isn't mom
There are no tests and the primary user can't report bugs precisely. The only
real safety net is the developer (the user) smoke-testing on her own phone
before mom wakes up to a broken app. Before any push, ask the user to run this
2-minute check and wait for her confirmation: PIN in → voice-record one expense
→ play one English sentence → open album. Encode her answers; never substitute
your own "it should work".

## Part 3 — Degradation forecast: where this system rots first, and the counters

1. **Docs drift from code.** Section names, behaviors, and line counts change;
   stale docs are worse than none. Counter: docs reference grep-able banner
   names, never line numbers; `~/.claude/docs/maintenance.md` makes fixing
   stale references a no-permission-needed edit — do it the moment you notice.
2. **Satellite docs stop being read.** A future session sees CLAUDE.md, skips
   the table, and re-derives everything badly. Counter: every hard rule lives in
   CLAUDE.md itself (the satellites only add detail and procedure); the table
   gives concrete read-triggers, not "see also".
3. **lessons.md becomes noise.** Append-only logs bloat until nobody reads them.
   Counter: `~/.claude/docs/maintenance.md` sets a hard compaction trigger
   (~15 entries) and requires user sign-off to compact, so it can't silently
   vanish either.
4. **The confirm-before-coding rule erodes.** Under time pressure the user says
   "直接改就好" a few times and future sessions generalize that into a norm.
   Counter: treat each such instruction as scoped to that one request; if the
   user wants the rule gone, propose editing CLAUDE.md rule 2 explicitly —
   that's the legitimate path (maintenance.md requires her approval for it).

## Part 4 — Honest ceiling: what this system cannot do

- **It fixes execution, not taste.** Checklists catch syntax errors, shrunken
  fonts, and self-certified work. They cannot decide whether mom would prefer
  「記帳」or「花費」as a label, or whether a feature is worth building. For those:
  ask the user, ideally with 2–3 concrete options (AskUserQuestion supports
  preview mockups). Never resolve a taste question by rubric.
- **Ambiguous requests stay ambiguous.** No template extracts a requirement the
  user hasn't formed yet. The counter is CLAUDE.md rule 2 (restate and confirm),
  not cleverness.
- **Model ceilings are real.** If Sonnet fails twice and Opus fails twice on the
  same subtask, more retries won't help. Say so plainly, present the failure log,
  and let the user decide: simplify the ask, try `fable` if available, or drop it.
- **Unverified facts in these docs** (marked UNVERIFIED throughout): mom's exact
  device/browser; Supabase RLS status; whether the `fable` model override remains
  available after 2026-07-07. Verify before relying; never present these as known.
