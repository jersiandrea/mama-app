# Delegation templates

Copy the template, fill every `<slot>`. If you cannot fill the acceptance
criteria, you are not ready to delegate — figure out what "done" means first.
Remember dispatch.md's calibration: in this one-file repo, most work should NOT
be delegated; these templates matter most for research and fresh-eyes review.

## 1. Search / locate
> Find <what> in <where/scope>.
> Context: I need this in order to <why>.
> Report: for each hit, `file:line` + one line on what surrounds it. If nothing
> is found, list exactly what patterns and paths you searched so the negative
> is trustworthy.
> Acceptance: I can jump straight to the code without searching again.

Agent: `Explore`, breadth "quick" for a named symbol, "medium" for a concept.

## 2. Implementation
> Implement: <precise behavior, including the Chinese UI strings to use>.
> Context: <which banner section of index.html; what the user approved>.
> Constraints: CLAUDE.md rules 1/4/5 apply verbatim — no font/tap-target
> reductions, ES5 style only (`var`, `function`, string concat), no new files,
> Traditional-Chinese UI text. Place code under the <SECTION> banner.
> Acceptance: the judgment.md §2 syntax check passes (paste its two commands
> and how to read the output); `git diff` touches only <expected area>; every
> failure path shows a visible Chinese message.
> Report: what changed + `file:line` per hunk + the syntax-check output.

Agent: `general-purpose`, model `sonnet`.

## 3. Refactor (behavior-preserving)
> Refactor <target> to <goal>, preserving behavior exactly.
> Context: <why this refactor is worth it — if you can't say, don't do it>.
> Constraints: same as Implementation. Additionally: no output/UI change of any
> kind; keep function names callable from inline `onclick=` handlers in the
> HTML (they must stay global).
> Acceptance: syntax check passes; a listing of every call site of the touched
> functions (grep output) showing each still resolves.
> Report: summary + call-site grep evidence + syntax-check output.

Agent: `general-purpose`, model `sonnet`. A separate fresh reviewer confirms
behavior preservation (template 5) before the change is accepted.

## 4. Research (web / docs)
> Question: <one precise question, with the decision it will inform>.
> Constraints: prefer primary sources (MDN, caniuse, supabase.com docs).
> Include URLs. Explicitly separate "found in a source" from "I inferred".
> Report: answer in ≤15 lines + source URLs. Anything longer goes to a file in
> the scratchpad; return the path.
> Acceptance: the answer names browsers/versions/limits specifically —
> "generally supported" is a FAIL.

Agent: `general-purpose`, model `sonnet`.

## 5. Review / verify (fresh eyes — never the author)
> Review <diff or file paths> against this checklist: <paste the relevant
> items from docs/judgment.md — do not just link them>.
> Context: none beyond this, deliberately. Judge only what you can see and run.
> Report: PASS/FAIL per item with evidence (`file:line` or command output).
> "Looks fine" is not evidence. End with the single biggest risk you see, even
> if every item passes.
> Acceptance: every checklist item has explicit evidence attached.

Agent: `general-purpose` (must be fresh — no prior context), model `sonnet`;
use `opus` if the diff touches PIN, deletion paths, or accessibility CSS.
