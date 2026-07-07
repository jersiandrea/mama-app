# Judgment rubrics

These externalize decisions that otherwise need a stronger model. Each rule has
a PASS example (do this) and a FAIL example (don't). When a rubric and a user
instruction conflict, the user wins — but say out loud that they conflicted.

## 1. When to escalate to a stronger model

Escalate when ANY of these holds:
- The same subtask failed twice (with genuinely different attempts).
- The change requires reasoning across 3+ sections of index.html at once
  (e.g. reworking the `_pt` speech-cancellation scheme, which threads through
  playWord/playVocab/stopAll).
- The user reports breakage with no repro, and 15 minutes of reading hasn't
  localized it.

Exception: `haiku` escalates after a **single** failure (dispatch.md's ladder —
don't debug haiku). The "failed twice" trigger applies to sonnet and above.

PASS: Sonnet twice produced a broken edit to `playWord()`'s callback chain →
stop, escalate to opus with both failed diffs and the observed misbehavior.
FAIL: escalating because a Chinese UI string "feels nuanced" — copy edits never
need escalation; ask the user instead (§3).
FAIL: quietly making a third attempt that's a small variation of the second.

## 2. Definition of done

A change is done only when ALL of these are true:
- [ ] `git diff` re-read top to bottom; you can justify every hunk.
- [ ] Inline JS still parses. Run:
      ```
      awk '/^<script>$/{f=1;next} /^<\/script>$/{f=0} f' index.html > /tmp/app_check.js
      osascript -l JavaScript /tmp/app_check.js 2>&1 | head -2
      ```
      Reading the output: `SyntaxError` = you broke it, fix before anything
      else. `execution error` with TypeError/ReferenceError (about console/
      document/supabase) = **parse succeeded**; that's the expected PASS state,
      since there's no DOM outside a browser. (Verified working 2026-07-07;
      this machine has no node.)
- [ ] No font-size decreased, no tap target shrunk, no genuinely new ad-hoc
      color. Check: `git diff | grep -iE "font-size|padding|width|height|min-width"`
      and compare old→new values by hand. Some controls are sized by
      width/height, not padding (`.mic` 76px, `.tgl` 52×28, `.wplay`) — eyeball
      those whenever the diff touches CSS.
- [ ] The new path fails gently in Chinese (visible message, e.g.
      `st.textContent='...請再試一次'`) — consistent with the section's pattern.
- [ ] The user confirmed the behavior matches what she approved (CLAUDE.md
      rule 2) — on her phone, if the change is user-visible.

PASS: after an accounting edit — syntax check run, diff shows only the
ACCOUNTING section changed, user replied「對，就是這樣」to a behavior summary.
FAIL: declaring done because the Edit tool call succeeded.
FAIL: "tests pass" — there are no tests here; that sentence is a hallucination.

## 3. When to stop and ask the user

This list adds specific triggers on top of CLAUDE.md rule 2's blanket
restate-and-confirm — it does not narrow rule 2. Ask — once, with questions
batched, offering concrete options (AskUserQuestion supports preview mockups)
— when:
- The decision is about mom's preference: wording, reminder times, category
  names, colors, layout.
- The change deletes data or modifies any delete path.
- It touches the PIN, Supabase URL/key, or the `ai-proxy` request shape.
- You're about to push (always, every time, no standing authorization).
- Two readings of the request lead to different code.

PASS: 「『記帳提醒改晚一點』— 改成 21:00 還是 21:30？首頁上顯示的『每天晚上 8:30』
文字要一起改嗎？」 (one message, both questions, concrete options)
FAIL: asking "should I proceed?" after every mechanical step.
FAIL: the mirror image — silently picking 21:00 and pushing.

## 4. Wrong-direction signals — change approach, don't retry harder

- Your plan adds a dependency, build file, framework, or new code file →
  forbidden by CLAUDE.md rule 4. The plan is wrong by definition; no retry.
- A "small" fix now touches 3+ banner sections → re-read docs/architecture.md;
  a narrower seam almost certainly exists.
- You are fixing your own fix for the second time → `git checkout index.html`
  (or `git stash`) back to the last good state, write what you learned into
  docs/lessons.md, then re-approach from the architecture doc.
- You're weakening a safety mechanism to make a feature work (removing the AI
  timeout, raising the 10s/5MB video caps, skipping `esc()` on interpolated
  strings) → the mechanism is load-bearing; find another way or ask the user.
- The AI proxy keeps failing → the app is designed to degrade (local parsing,
  canned replies). Don't "fix" reliability by making the app depend harder on
  the proxy.

PASS: second failed patch to video playback → revert, log lesson, re-read the
ALBUM section, notice the overlay resets `src` deliberately, take the new seam.
FAIL: attempt #3 with more code piled on attempts #1–2 still in the file.

## 5. Minimum quality bar for any merged change

- Matches surrounding style exactly: `var`/`function`, string concat, section
  placement under the right `════` banner.
- All user/db strings interpolated into innerHTML pass through `esc()`.
- Failure paths visible in Chinese; success paths announced the way neighboring
  features do (e.g. `sayZh()` confirmations in ACCOUNTING).
- UI additions: primary/actionable text ≥19px; secondary/meta text may match
  the app's existing smaller patterns (13–18px, e.g. `.dt`, `.soft`) but never
  go below the closest existing analog. Tap targets comparable to existing
  buttons. Colors from the `:root` palette, or matching an existing hardcoded
  value used the same way (e.g. `CAT_COLOR` swatches) — never a genuinely new
  ad-hoc color.
