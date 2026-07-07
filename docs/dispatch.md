# Model & agent dispatch

Environment facts, verified 2026-07-07 (if this file is old, re-check names
against the system prompt's agent list before relying on them):

- Agent tool `subagent_type` values: `general-purpose`, `Explore` (read-only
  search; takes a breadth hint: quick / medium / very thorough), `Plan`,
  `claude` (catch-all), `claude-code-guide` (questions about Claude Code
  itself), `statusline-setup`. `fork` inherits the full conversation.
- Agent `model` override values: `haiku`, `sonnet`, `opus`, `fable`.
  `fable` availability after 2026-07-07 is UNVERIFIED — if the Agent call
  rejects it, use `opus`.
- No MCP servers are configured. No node on this machine (affects verification
  commands — see judgment.md §2).
- Relevant skills: `/code-review` (diff review), `verify` (exercise a change
  end-to-end), `run` (launch the app), `security-review` (anything
  security-adjacent — read handoff.md Part 2 first for the known caveats).

## Calibration for THIS repo — read before delegating anything

The repo is one ~65KB file. Reading a section yourself (grep banner → Read
range) costs less than briefing a subagent to do it. **Default here: do NOT
spawn subagents for code reading or single-file edits.** The main conversation
does the work; agents are for:

1. **Web research** (browser API compatibility, Supabase behavior/limits) →
   `general-purpose` with instructions to use WebSearch/WebFetch.
2. **Independent verification of risky work** (fresh eyes — see below).
3. **Parallel batch work** — only if this project ever grows past a few files.

## The delegation contract (when you do delegate)

Every delegation prompt must contain all five, or don't send it:
(1) the goal, (2) why it matters / surrounding context, (3) constraints,
(4) acceptance criteria — how the agent knows it passed, (5) report format.
Fill-in templates: `docs/delegation-templates.md`.

Reports come back as conclusions + `file:line` references only. Any artifact
longer than ~30 lines goes into a file (scratchpad for throwaway, `docs/` for
keepers) and the agent returns the path, not the content.

## Escalation / de-escalation ladder

- `haiku` gets a subtask wrong once → redo it on `sonnet`. Don't debug haiku.
- `sonnet` fails the same subtask twice → escalate to `opus` (or `fable` if
  available), attaching the full failure log: what was tried, exact errors,
  what was ruled out. Never escalate without the log — the stronger model just
  repeats attempt #1.
- A solved pattern that now repeats mechanically → hand down to `haiku` in
  batches, with one worked example in the prompt.
- Hard cap: **two retry rounds per approach.** A third attempt must use a
  different approach, or go to the user with the failure log. judgment.md §4
  lists the signals that mean you're in wrong-direction territory (change
  approach entirely) rather than merely short of a good attempt.

## Verification — never self-certify

"Risky" = the diff touches the PIN gate, any data-deletion path, accessibility
CSS (font sizes, tap targets, colors), or is about to be pushed.

1. Spawn a FRESH `general-purpose` agent that has not seen your reasoning.
   Give it only: the file paths (or diff), the relevant checklist from
   `docs/judgment.md`, and "report PASS/FAIL per item with evidence
   (file:line or command output)".
2. Files you wrote: reviewer verifies by read-back — it reads from disk and
   compares against the stated intent, not against your summary.
3. Code: reviewer (or you) runs the syntax check and grep checks in
   judgment.md §2. Real behavior beats inspection: use the `verify` skill when
   there is a runtime surface to drive.
4. High-risk judgment calls: get a second opinion from a second fresh agent, or
   produce 2–3 candidate solutions and have one reviewer rank them with
   reasons. Reviewer model: `sonnet`; use `opus` when the diff touches
   PIN/deletion/accessibility.
