# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Primary user & non-negotiable UI constraint

The primary (real-world) user of this app is the developer's mother, who has poor eyesight. **Every UI change must preserve large fonts, high contrast, and large buttons — this is the highest-priority constraint and must never be compromised**, even for changes that seem like minor visual polish or "cleanup." Never shrink font sizes, tighten tap targets, or lower contrast (e.g. `--soft` text on `--card`/`--bg`) to fit more content or match a "cleaner" design — reflow or restructure instead.

## Workflow requirement

Before implementing any change to functionality, confirm your understanding of the request with the user first and get their go-ahead — do not start editing `index.html` immediately on request.

## Deployment

Deployment is push-to-deploy: pushing to the `main` branch on GitHub triggers GitHub Pages to auto-deploy `index.html` to https://jersiandrea.github.io/mama-app. There is no staging environment or CI test gate — anything pushed to `main` goes live immediately, so treat pushes to `main` with the same caution as a production deploy (confirm with the user before pushing, per the workflow requirement above).

## What this is

「媽媽的好幫手」(Mom's Helper) — a single-file, PIN-gated PWA-style web app for a family, in Traditional Chinese. It has no build step, no package manager, and no test suite: the entire app (markup, CSS, and JS) lives in `index.html` and is deployed by simply uploading/serving that file as a static site (commit history is literally "Add files via upload"). There is no local dev server config — open/serve `index.html` directly to work on it.

Backend is a Supabase project (`SB_URL`/`SB_KEY` hardcoded near the top of the `<script>` block) using the public/anon key directly from client-side JS, plus one Supabase Edge Function (`ai-proxy`, not in this repo) that proxies calls to an AI model for expense classification and English conversation replies.

## Architecture

Everything is one file, organized top-to-bottom into clear `════` banner-delimited sections. When making changes, find the relevant section rather than searching the whole file:

1. **PIN gate** — a hardcoded `PIN` constant gates the whole app via `sessionStorage`. There's no real auth; this is a shared-family passcode, not a security boundary.
2. **App init / tab navigation** — `initApp()` bootstraps everything on load; `gotoTab(name)` switches between the four tabs (`home`, `acc`, `eng`, `alb`) by toggling `.on` classes, and lazily reloads that tab's data.
3. **Accounting (記帳)** — voice-driven expense entry via Web Speech API (`SpeechRecognition`, zh-TW). Flow: mic captures a spoken sentence → `processAccVoice` calls the `ai-proxy` Supabase function (task `classify_expense`, 8s timeout) to extract `{amount, category, desc}` → falls back to local regex/keyword parsing (`extractAmt`, `guessCategory`, `CK` keyword map) if the AI call fails or times out → row inserted into the `accounts` table. Category pie charts are hand-rolled SVG (`renderPie`), not a charting library.
4. **English learning (英語學習)** — a hardcoded `ALL_WORDS` array of English/Chinese sentence pairs and a `VOCAB` list. `todaysWords()` uses a seeded PRNG (`seededRand`, seeded from the date) so every user sees the same 5 sentences on a given day without server state. Playback (`playWord`/`playVocab`) chains multiple `SpeechSynthesisUtterance` calls (zh → slow en → word-by-word en → zh) using a token-based cancellation scheme (`_pt`/`chk()`) to abort stale speech chains when the user taps again. The conversation feature (`toggleEng`) sends chat history to `ai-proxy` (task `english_chat`) with a canned fallback reply bank (`FB`) keyed by keyword-matched topic when AI is unavailable.
5. **Notifications** — local, per-day, `localStorage`-tracked (`nf_<type>_<date>` keys) browser `Notification` reminders scheduled with plain `setTimeout` from page-load time; there is no service worker or push, so a reminder only fires if the tab is open when the target time arrives.
6. **Album (相簿)** — photo/video sharing to the `album_posts` Supabase table. Images are downscaled/compressed client-side to a data URL before upload (`compressImg`, max 1600px / ~900KB, JPEG quality stepped down until under budget); videos are capped at 10s / 5MB and also stored as base64 data URLs (no Supabase Storage/CDN — media is inline in the table row). Posts carry an `expires_at` 24h TTL; `loadAlbum()` fetches non-expired posts and opportunistically deletes expired ones client-side on every load (no server-side cron).

## Data model (Supabase, inferred from client calls)

- `accounts`: `id`, `amount`, `category` (食/衣/住/行/育/樂/其他), `raw` (description), `profile` (who logged it), `created_at`.
- `album_posts`: `id`, `text_content`, `photo` (data URL or null), `video` (data URL or null), `profile`, `expires_at`, `created_at`.

There's no local schema/migrations file in this repo — schema exists only in the Supabase project itself.

## Conventions to preserve when editing

- Vanilla ES5-leaning JS (`var`, `function`, no modules/bundler, no framework). Keep new code consistent with this style rather than introducing build tooling.
- Every async/DOM-dependent call in `initApp()` is wrapped in its own `try/catch` with a `console.warn`/`console.error` — errors in one widget must not block the rest of the app from initializing.
- UI strings are Traditional Chinese and written for an older/less tech-fluent user (large fonts/buttons, explicit voice prompts like「點一下開始說話，再點一下停止」). Match this tone and simplicity in new UI text.
- AI calls always go through `callAI()` with a manual timeout race and a local fallback path — never assume the `ai-proxy` call succeeds or is fast.
