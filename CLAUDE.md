# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The picture in one paragraph

「媽媽的好幫手」(Mom's Helper): a Traditional-Chinese family app whose primary real-world user is the developer's mother, who has poor eyesight. The entire app is ONE file — `index.html` (~1,100 lines: CSS + HTML + vanilla ES5 JS) — backed by a Supabase project plus one Supabase Edge Function (`ai-proxy`, lives outside this repo). Pushing to `main` deploys it live to https://jersiandrea.github.io/mama-app via GitHub Pages — no CI, no staging, no tests.

## Non-negotiable rules

1. **Accessibility first, always.** Never reduce any font size, contrast, or button/tap-target size, for any reason (base font is 20px; big action buttons are 22px). If a change needs more room, reflow or restructure — never shrink or tighten.
2. **Confirm before coding.** Before implementing any functional change, restate your understanding to the user and get her go-ahead. (Edits to `docs/*.md` usually don't need this — except the cases listed in `docs/maintenance.md` § "Ask the user BEFORE".)
3. **Never `git push` without explicit authorization given in the current conversation.** A past session's go-ahead does not carry over. Push = instant production deploy to mom's phone. Local commits are fine when asked.
4. **No new tooling, ever.** No package.json, no build step, no framework, no TypeScript, no splitting `index.html` into modules or extra JS/CSS files. All app code stays in `index.html` in the existing style: `var`/`function`, string concatenation; `async`/`await` and Promises are already used there and are fine — do not "downgrade" them. No arrow functions, template literals, classes, or modules. Markdown under `docs/` is the only sanctioned place for new files.
5. **UI text is Traditional Chinese**, warm and simple, written for an older non-technical user (e.g.「點一下開始說話，再點一下停止」). Match that voice in any new copy.

## Working in index.html

Do not read the whole file to find things. The JS is divided by banner comments; locate sections with:

```
grep -n -A1 "════" index.html
```

(The section name sits on the line *after* the first `════` border — without `-A1` you get bare line numbers and no names.) Sections in order: CONFIG · PIN GATE · APP INIT · TAB NAVIGATION · UTILITIES · VOICE HELPERS · AI CALL · ACCOUNTING · ENGLISH LEARNING · NOTIFICATIONS · ALBUM. Read only the range you need. Structure, data model, and quirks: `docs/architecture.md`.

After editing JS, run the syntax check in `docs/judgment.md` §2 (this machine has no node; the osascript pipeline there is tested and works).

## When to read the other docs

| Situation | Read first |
|---|---|
| Any non-trivial change to app behavior | `docs/architecture.md` |
| Spawning a subagent / choosing a model | `docs/dispatch.md` |
| Unsure whether it's done, whether to ask the user, whether to escalate, or whether you're off track | `docs/judgment.md` |
| Writing a delegation prompt | `docs/delegation-templates.md` |
| You made a costly mistake, or want to edit these docs | `docs/maintenance.md`, then log it in `docs/lessons.md` |
| Starting a large task — context from the session that wrote these docs, incl. security caveats | `docs/handoff.md` |
