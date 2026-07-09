# Architecture — index.html in detail

Everything lives in `index.html`: a `<style>` block, the HTML (PIN gate, `#app`
with four `.tab` panes + bottom `nav`, modals `edit-modal`/`del-modal`, photo and
video overlays), and one `<script>` block divided by `════` banner comments.
Find any section with `grep -n -A1 "════" index.html` (the name sits on the
line after the border, so `-A1` is required) — never navigate by line numbers
(they rot); navigate by banner name and function name.

## Backend

Supabase (URL + publishable key hardcoded in the CONFIG section, used directly
from the browser) plus one Edge Function `ai-proxy` — **not in this repo**; it
proxies to an AI model. Client contract: `sb.functions.invoke('ai-proxy',
{body:{messages, task}})` where `task` is `'classify_expense'` or
`'english_chat'`; response mimics the Anthropic Messages shape (`data.content[]`
with `{type:'text', text}` — see `callAI()`).

## The script sections (banner names)

1. **CONFIG** — `SB_URL`, `SB_KEY`, `PIN`, supabase client `sb`, `CAT_COLOR` map.
2. **PIN GATE** — `checkPin()`; a shared family passcode held in
   `sessionStorage` key `ok`. It is a UI courtesy, not a security boundary.
3. **APP INIT** — `initApp()`: every widget init is wrapped in its own
   `try/catch` so one failure can't block the rest. Preserve this pattern.
4. **TAB NAVIGATION** — `gotoTab(name)` toggles `.on` classes for
   `home|acc|eng|alb` and lazily reloads that tab's data.
5. **UTILITIES** — `esc()` (HTML-escape — use it on ALL user/db strings you
   interpolate into innerHTML), `fmtDt`, `closeModal`, `fileToB64`. Known gap:
   `loadSpend()`'s row template interpolates `e.category` unescaped even though
   it can come from the AI proxy's JSON — wrap it in `esc()` if you touch that
   line, and never copy the unescaped pattern into new code.
6. **VOICE HELPERS** — `tts()`, `getSR(lang)` (wraps `webkitSpeechRecognition`),
   `micErr()` (Chinese error messages for mic failures).
7. **AI CALL** — `callAI(messages, task)`: 5s `Promise.race` timeout, returns
   `null` on any failure. Callers MUST handle `null` with a local fallback.
8. **ACCOUNTING** — voice sentence → `processAccVoice`: tries `ai-proxy`
   (`classify_expense`, extra 8s race) to get `{amount, category, desc}`, falls
   back to local parsing (`extractAmt` handles digits AND Chinese numerals via
   `cnToNum`; `guessCategory` keyword map `CK`). Inserts into `accounts`.
   Charts are hand-rolled SVG pies (`renderPie`) — no chart library.
9. **ENGLISH LEARNING** — hardcoded `ALL_WORDS` (100 sentence pairs) + `VOCAB`.
   `todaysWords()` picks 5/day with a date-seeded PRNG (`seededRand`) so all
   devices agree without server state. UI is a single flashcard deck
   (`buildDeck()` = 5 sentences + today's vocab in one array, `_deckIdx`
   cursor, `renderCard()`, tap left/right half of `#flashcard` = `nextCard()`
   prev/next — swipe was removed because it collides with Safari's
   back/forward edge gesture; don't reintroduce it). `playCard()` plays the current
   card: sentence sequence zh → en 0.8x → word-by-word en 0.8x (2s gaps) → zh;
   vocab sequence zh → en → en → zh. Rate is 0.8 (was 0.5 — 0.5 slurred on
   iOS; do not lower it again). Cancellation is token-based: `_pt` increments
   on `stopAll()`, every async step checks `chk()` before proceeding. If you
   touch playback, keep the token pattern or overlapping audio returns.
   Conversation (`toggleEng`) sends `chatLog` to `ai-proxy` (`english_chat`)
   with canned fallback bank `FB`.
10. **NOTIFICATIONS** — browser `Notification` scheduled by `setTimeout` from
    page load, deduped per-day via localStorage `nf_<type>_<date>`. No service
    worker: reminders fire only if the tab is open at the target time. This is a
    known limitation, not a bug — tell the user before "fixing" it.
11. **ALBUM** — four screens inside `#t-alb`, toggled by `showAlbScreen()`
    (`home` big-plus entry → native camera via one `<input capture
    accept="image/*,video/*">` → `preview` cancel/upload → `viewer` → `chat`).
    `viewer` is a true full-screen `position:fixed` panel (`inset:0`,
    z-index 150 — it deliberately covers the bottom nav; closing it reveals
    the nav again). `chat` stops 76px above the nav so the nav stays visible
    there. Both live inside the tab div so tab switching hides them. Viewer
    (`renderViewer`/`openViewer`) shows one post full-screen, newest first:
    uploader + `fmtDt` top-left, download/chat buttons bottom, videos loop,
    tap right half = older, tap left half = newer (past the first post =
    back to `home`; swipe was removed — Safari edge-gesture collision).
    `albCapture()` force-stops all speech playback/recognition before
    opening the camera input — without this iOS shows「通話時無法錄影」
    because the page still holds an audio session; keep that cleanup. Text-only posts from the old UI
    still render (`.av-text`). Chat (`openChat`) is per-post: rows in
    `album_chats` keyed by `post_id`, text only, expires with the post,
    polled every 8s while open (`chatTimer`). Media pipeline unchanged:
    `compressImg` (canvas, max 1600px, JPEG stepped down to ~900KB), videos
    capped 10s / 5MB, stored as **base64 data URLs inline in the row** (no
    Storage bucket). Posts get a 24h `expires_at`; `loadAlbum()`
    opportunistically deletes expired `album_posts` AND `album_chats` rows
    client-side — there is no server-side cleanup.

## Data model (Supabase; inferred from client calls — no schema file exists)

- `accounts`: `id` ('acc_'+Date.now()), `amount` int, `category`
  (食/衣/住/行/育/樂/其他), `raw` (description), `profile` (which family member),
  `created_at`.
- `album_posts`: `id` ('alb_'+Date.now()), `text_content`, `photo` (data URL |
  null), `video` (data URL | null), `profile`, `expires_at`, `created_at`.
- `album_chats`: `id` ('chat_'+Date.now()), `post_id` (→ album_posts.id, no
  FK), `profile`, `text_content`, `expires_at` (copied from the post),
  `created_at`.

## Conventions to preserve

- Style: `var`, `function`, string concat. `async`/`await`, Promises, and
  newer methods (`padStart`, `includes`, `Object.assign`) are already used
  throughout and are fine — don't "downgrade" them, and don't label the file
  ES5. No arrow functions, template literals, classes, or modules; no new
  `let`/`const` (a few `const`s exist in CONFIG). Match the surrounding code
  exactly.
- Every user-facing failure sets a visible Chinese message (e.g.
  `st.textContent='儲存失敗，請再試一次'`) — never a silent catch.
- AI calls always run under a timeout race with a working local fallback.
- localStorage keys in use: `who`, `pc_Y_M_D` (play count), `notif-eng`,
  `notif-acc`, `nf_<type>_<dateString>`. sessionStorage: `ok` (PIN passed).
- Likely client is iPhone Safari (webkit speech APIs, `-apple-system` font) —
  UNVERIFIED; prefer feature detection over UA assumptions, as the code already
  does.
