# Dr Omni AI

Clinical decision-support + medical-education assistant for patients, medical students and doctors.
Live page: `https://perchance.org/<generatorName>` (see `window.generatorName`).

## Project layout

- `main.pjs` — `$meta` (title/description/image/tags, minimal header) + two plugin imports:
  `generateText = {import:ai-text-plugin}` and `superFetch = {import:super-fetch-plugin}` (used by the
  deep reference search to fetch site pages/SERPs cross-origin). Keep lists/config here; almost all
  logic lives in `index.html`.
- `index.html` — the WHOLE application (single file, ~2200+ lines): styles, markup, all JS.
  Do **not** add `<html>/<head>/<body>` tags — it is body content only, served inside an iframe.
- `src/README.md` — this file.
- `src/YemenMd_tables/*.csv[.gz]` — the Yemen medical-guide data (see the Directory section).
- `src/WorldHospitals/hospitals.csv.gz` — the **global** hospital/medical-centre database (22 008
  facilities across 188 countries), served to the Directory tab for every country except Yemen.
- `src/app.js`, `src/engine.js` — **unrelated leftovers** from an earlier audio/song idea, not
  referenced by `index.html`. Safe to delete (flagged to the user, awaiting confirmation).

`scratch/` is ephemeral session space.

### Logo / icon (شعار التطبيق)

The official logo (rounded white app-icon tile: teal→blue→violet gradient medical cross with a white
ECG heartbeat line) is hosted at `https://user.uploads.dev/file/518c250165765ddb567c66c5b8dbd2bf.png` (512×512 PNG) and is
referenced by the const `LOGO` in `index.html` (the hero image on the **language-splash** screen, the
logo embedded into exported Word docs, the **header brand icon** directly beside the app name, and the
favicon/apple-touch-icon).

#### Brand name — user rule (must hold)

The app is called **`Dr Omni AI`** (Latin script, no translation, `dr` in the user's original request).
The `🩺` emoji that used to precede `OmniDoctor AI` was removed from the language-splash title, the
memory-card greeting and the Word-doc title — **do not re-add an emoji/sticker to those text strings**.
The user later asked for the **brand logo image** to sit directly beside the header app name (that one
image is wanted; emojis/stickers elsewhere are not). All locale `appName` and chat-`doctor`
fields are set to the untranslated brand `Dr Omni AI`; the localized description stays in `tag` /
`docTag`. The splash title and the Word-doc header each show the brand exactly once (the old duplicate
localized-name line / `<p>` was removed), so don't re-add a second name line. The previous brand was
`OmniDoctor AI` / localized "Miracle Doctor"/"الطبيب المعجزة" — that name is gone everywhere.
The **favicon and apple-touch-icon** reuse that exact same URL (single asset; browsers downscale it).
Favicon links are injected into `<head>` by a tiny inline script at the top of `index.html` (plus static
`<link>` tags). `main.pjs` `$meta.image` uses the same logo URL for the listing/social card.
To swap the logo, produce a square 512px PNG with a rounded-corner alpha mask, upload it, and replace the
URL everywhere (`grep -n "uploads.dev/file"`).

## UI theme (logo-derived palette)

All colors come from the logo (teal → royal blue → violet shield). The `<style>` block starts with a
`:root` palette and the rest of the sheet uses those vars — **edit the palette there, not scattered hexes**:

| var | value | use |
|-----|-------|-----|
| `--teal` | `#00a3a3` | logo teal (mic button, secondary accents) |
| `--blue` | `#2a6fd6` | logo royal blue (camera button, links) |
| `--violet` | `#8a3ffc` | logo violet (📎 file button) |
| `--brand` / `--brand-d` | `#1e5fce` / `#17419e` | headings, active chips, primary text |
| `--grad` | `linear-gradient(135deg,#00a3a3,#2a6fd6 52%,#8a3ffc)` | header, primary buttons, memory card |
| `--grad-2` | `linear-gradient(120deg,#00a3a3,#2a6fd6)` | user chat bubbles, table heads |
| `--tint` / `--tint-2` | `#eef3ff` / `#e2ecff` | chip & soft-panel backgrounds |
| `--line` / `--line-2` | `#dbe5fb` / `#c8d9fa` | borders |
| `--ink` | `#0f2557` | body text |

Semantic colors stay off-palette on purpose: emergency red (`.erCard`, `.dangerBtn`, `.box`),
warning amber (`.disclaimer`, type badges) and correct-green (`.opt.right`, `.pill`).
Exported Word docs (`downloadWordDoc`) use the same `#2a6fd6` header color hardcoded (no CSS vars there).

### Header layout

`header` is a **two-tier flex column** at every width:

- `.hRow` — `justify-content:space-between;direction:ltr` → `.brand` (`<img>` logo + "Dr Omni AI" text)
  pinned on the **LEFT**, and `#settingsBtn` (`.iconbtn` gear) on the far **RIGHT**. The explicit
  `direction:ltr` is required because the app is RTL by default (`document.documentElement.dir = "rtl"`),
  which would otherwise send the brand to the right; forcing LTR on this row keeps brand=left / gear=right
  in **both** RTL and LTR, so verify in both languages after any change. Don't move the settings button
  back into `.brand` or into `.tabs`.
- `.tabs` — the 6 tab buttons (`#tabBar`): one scrollable centered row on ≥601px, and a
  `grid-template-columns:repeat(3,1fr)` **3×2 grid** at ≤600px so all six are visible without scrolling.
- `--hdr` is set by `fitHeader()` (on load/resize/orientationchange) to `header.offsetHeight + 14px`
  and positions `#memoryCard` just under the header.

## Features (tabs)

| Tab | id | What it does |
|-----|----|--------------|
| 💬 Chat | `chatTab` | Free AI chat + voice (TTS) + image analysis + **any-file upload**. |
| 🧠 MCQ | `mcqTab` | **Endless** test: 15-question rounds, a score screen after each; the AI keeps inventing new non-repeating questions. |
| 🏥 Directory | `dirTab` | **Global** medical directory: a country selector (188 countries) over `src/WorldHospitals/hospitals.csv.gz` for every country, plus the full 12-sub-tab Yemen guide when اليمن is selected. |
| 💊 Drugs | `drugTab` | Global multilingual drug search (RxNorm names/class + FDA/DailyMed label sections, Arabic-translated), with a hard **banned-substance** filter (narcotics/sedatives/high-harm). |
| 🔗 References | `refsTab` | Global reference library + live search (see below). |
| 📚 Memory | `memTab` | Doctor's memory: **patient files** (per-person transcript + reports), saved pearls/lessons, profile. |

### Chat layout & attachments

- The chat input bar (`.inputRow`) is pinned to the bottom of the screen: `#chatTab.on`
  is a flex column, `#chatBox` is `flex:1;overflow-y:auto`, and the input row is the last
  flex child. Do not reintroduce `position:sticky` there.
- Input row buttons: `📎 fileBtn` (any file), `📷 camBtn` (camera/gallery → `handleImageFile`),
  `🎤 micBtn` (speech-to-text), `➤ sendBtn`.
- **Pending attachments (multi-upload)**: `#imageInput` and `#fileInput` are `multiple`, and picking
  files does **not** send them — `onPicked`/`onPickedFile` push them into `pendingAttach` and
  `renderPending()` shows removable chips (with image thumbnails via `URL.createObjectURL`, ✕ =
  `removePending`) in `#pendingBar` above the input row. `onSend()` sends nothing until the user
  presses ➤ (or Enter): it takes `pendingAttach`, clears the bar, and runs `processAttachments(atts, text)`
  which awaits `handleImageFile`/`handleAnyFile` **one file at a time** (the ai-text-plugin serialises
  requests, so never fire them in parallel). If the user also typed text, the text is shown once as a
  user bubble and passed as the `question` argument to the first attachment's analysis
  (`handleImageFile(file, isFlow, question)` / `handleAnyFile(file, question)`) so the answer addresses
  it. `resetChat()`/`clearChat()`/`startFlow()` clear pending.
- `#fileInput` has **no `accept`** (any type). `handleAnyFile(file)` routes by type:
  images → `handleImageFile`; **PDF** → `extractPdfText` (pdfjs-dist 3.11.174 via esm.sh, worker
  as a Blob URL from jsdelivr); **DOCX** → `extractDocxText` (mammoth via esm.sh);
  anything else → `readMaybeText` (plain text, rejects binary by control-char ratio).
  Extracted text is sent to the AI with `TASK_FILE`. Unreadable types show `file_unsupported`.
- **Files are NOT auto-saved to the patient file.** `TASK_IMG`/`TASK_FILE` instruct the model to
  begin its reply with `FILE_RELEVANT: yes|no`; `stripRelevance(out)` removes that marker and
  returns `{relevant, body}`. The reply bubble then gets `fileMsgActions(aiText, fileLabel, relevant)`
  (`handleImageFile`/`handleAnyFile`): a **💾 حفظ في ملف الحالة** button when relevant (or unknown),
  or a **حفظ رغم ذلك** button (with a confirm dialog + `file_unrelated` warning, `relevant===false`)
  when the model judged the file unrelated to this patient/case. `patLog` for the file + reply only
  happens when the user presses one of those buttons, so unrelated files never enter the patient file.


## AI pipeline

`askAI(system, user, image, opts)` → `root.generateText({instruction, onChunk})` (ai-text-plugin).
Pass `opts.onChunk(fullTextSoFar)` to **stream** the answer; `aiReply` and `finishFlow` stream into
the assistant bubble (a blinking `.cur` cursor while generating, replaced by the final `renderAI`
output + action buttons on finish). Don't revert to a blank wait — generation can take ~30s.
- **NO API KEYS ANYWHERE.** The engine is the perchance `ai-text-plugin` (`root.generateText`) plus
  `super-fetch-plugin` — both keyless. Never add a key prompts, key inputs, or "check your keys"
  wording to any UI/error string (users read that and think they must obtain a key). The generic
  failure string is `t("error_ai")` = "the AI engine couldn't be reached (no keys are needed), check
  your internet and try again".
- `askAI` retries up to `opts.retries` (default 3) with 0.8s×attempt backoff, so a transient engine
  hiccup self-heals instead of surfacing an error bubble.
- The ai-text-plugin serialises requests per user (extra calls queue and can appear to "hang" for
  minutes), so `aiBusy`/`aiBusyOn()`/`aiBusyOff()` gate the send path: `onSendText` and the file
  handlers refuse to start a second generation while one is running, showing `t("busy_wait")`.
  `aiBusyOn` also arms a 180s watchdog that clears the flag so a stalled generation can't lock the
  input forever. `maybeAutoLearn` defers while busy (never competes with the user's turn).
- `systemPrompt(task, ctx)` builds the system prompt. Order matters for prefix caching: the STATIC
  blocks (persona, IMPORTANT, TASK, EVIDENCE, OUTPUT RULES) come FIRST, then the per-turn context
  (`ctx.refs`, `ctx.live`, `ctx.kb`, `ctx.mem`, `ctx.prof`). Keep it that way (static text is
  re-read from cache; only the tail is uncached).
  Blocks: `ctx.refs` (CONNECTED GLOBAL REFERENCES) + `ctx.live` (LIVE REFERENCE DATA) + `ctx.kb`
  (local RAG banks) + `ctx.mem` (memory) + `ctx.prof` (profile) + OUTPUT RULES.
  Two **mandatory** instructions live in the static part (before the per-turn context, so they stay
  cached): **FIRST AID** (every answer must include a short 🚑 الإسعاف الأولي section with the
  immediate step-by-step actions, what NOT to do, and the exact moment to call emergency services/go
  to the ER) and **NEAREST CARE** (name the closest suitable doctor/hospital to the patient's home,
  with area + distance, and say whether to call ahead). `ctx.nearby` (a trailing block) carries the
  pre-computed nearest-care list — see below.
- **Nearest care / residence** (`residence()` + `nearbyContext()`): reads the active health file's
  `resCountry`/`resCity`/`resDistrict`/`lat`/`lon` (falling back to the flow's country). For Yemen it
  uses the YemenMD establishments + crews (distance-sorted, nearest doctors/hospitals/clinics with
  phone + maps link); for any other country it distance-sorts `hospitals.csv.gz`, or — with no GPS —
  states the country's hospital count and asks for the city. Memoised for 10 min (`_nearCache`).
  `freeChat`, `finishFlow` (TASK_ASSESS) and the file/image handlers pass `nearby:nearby` (fetched
  in parallel with `liveRefsContext`).
- `retrieveKB(text)` keyword-matches the `KB` array (local knowledge banks) → RAG context.
- Tasks: `TASK_CHAT`, `TASK_ASSESS` (7-step clinical checklist), `TASK_IMG`, `TASK_DRUG_REVIEW`.

### Reference connectivity (the 🔗 system)

- `REFS` — catalog of ~116 global references across 9 categories (`REF_CATS`): clinical,
  drugs, anatomy, evidence, guidelines, labs, imaging, calculators, regional (Yemen/EMRO).
  Each entry: `{c,category, n:name, u:deep-link template with {Q}, q?, ar,en}`.
- `ANATOMY` — 10 body systems `{ar,en,t}` with deep links (TeachMeAnatomy, Kenhub, Radiopaedia,
  Gray's, BioDigital, etc.).
- `collectLive(term, budgetMs)` — one aggregated live fetch: Wikipedia, openFDA drug label, RxNorm,
  PubChem, PubMed (eutils), Europe PMC, ClinicalTrials.gov, run in parallel with a PER-SOURCE
  `budgetMs` race (default 9s; `liveRefsContext` uses 2.5s) so one slow API can't stall the rest.
  Returns `{wiki,fdalabel,rxnorm,pubchem,pubmed,epmc,trials}`. Degrades gracefully.
- `liveRefs(term)` — renders `collectLive` results into `#liveRes` (used by the refs tab search).
- `liveRefsContext(term)` — same data as a compact text block for the AI prompt. Capped at 2.5s and
  memoised per term (`_liveCtxCache`, 10 min) so it adds minimal latency to a chat turn.
- `referenceContext(text)` — returns the canonical reference NAMES relevant to a query
  (keyword-routed) for the "CONNECTED GLOBAL REFERENCES" prompt block.
- `refSearchTerm(txt)` — strips EN/AR stopwords, takes up to 6 meaningful words.
- `openRefsFor(term)` — opens the refs tab and runs a live search for a raw term (kept as a general
  helper; the per-answer button now uses `openRefsFromReply` below).
- `refsFromReply(text)` / `openRefsFromReply(text, caseText, terms)` — **the per-answer 🔗 button
  searches the refs box for the DISEASES/CONDITIONS the DOCTOR mentioned in his reply, never the
  user's own words** (user rule): `refsFromReply` finds the trailing `🔎 المراجع:` (any language)
  header and splits the cited reference names (→ `renderMentionedRefs(names, caseQ)`), and
  `replyTermsMeta(text)` (line-based parser) strips + returns the hidden machine-readable
  **`🔍 TERMS: term1, term2, …`** line that every reply must end with.
- **`🔍 TERMS:` contract** — the `OUTPUT RULES` block of `systemPrompt()` instructs the model to make
  the very last line of every reply a hidden `🔍 TERMS: …` line holding **3-5 ENGLISH/LATIN search
  terms** for the diseases/drugs/symptoms the doctor just discussed. `replyTermsMeta` removes it from
  the displayed/streamed/copied/stored/TTS text (call sites: `aiReply`, the streaming `onChunk`,
  `finishFlow` after the PEARL strip, `fileMsgActions`, `handleImageFile`, `handleAnyFile`) and
  validates the terms (`/^[A-Za-z][A-Za-z0-9 '\-.]+$/`, ≤42 chars, deduped, max 6). Handles `🔍 TERMS:`,
  `**🔍 TERMS:**`, bare `TERMS:`; a reply without the line is left untouched. Real effect: a reply to
  an Arabic complaint yielded `hypertension, khat, headache, dizziness, sympathomimetic` and the refs
  box filled with `headache dizziness polycythemia khat` → real on-topic Wikipedia/PubMed hits.
- **Fallback path (no terms line)** — `buildReplyQuery(text)` → `replyQuerySource` drops the
  `🔎 المراجع:` footer and the trailing ⚠️ disclaimer line, adds the reply's emoji section headings,
  `fetchTranslate` renders it in English (up to 1000 chars), `refQueryFromLatin` strips stopwords
  (shared `QUERY_STOP`) and keeps ≤16 tokens → shown in `#refsMentioned` via
  `renderMentionedRefs(names, caseQ)`. Then `buildReplyQuery` scans the translated text for strong
  medical words and, if ≥2 exist, replaces the query with those ≤5 terms (`medStrong` first, KB
  `medVocab` next); `shortTopicQuery(caseQ,4)` reduces it to ≤4 clinical words for the input box +
  `liveRefs` (the live APIs — Wikipedia/PubMed/Trials — return far more hits for a short term than a
  sentence). `shortTopicQuery` takes the leading words, stops at the first `-ed/-ing` token after 2
  words (kills translation verb noise like "…reserved libido"), drops filler
  (`based/mentioned/according/options/overall/…`), and swaps a weak modifier (`severe/mild/…`) for a
  *medically strong* token found in a 30-word window — `medStrong` = `MED_WORDS` (a ~200-entry
  embedded English clinical vocabulary: conditions, symptoms, drug classes) or a disease suffix
  (`-itis/-osis/-emia/-pathy/…`, `syndrome`, `sepsis`, …) or a `medVocab` (lazily built from
  `KB[].kw`) word. Output is re-ordered so medical terms come first.
  Real effect: fallback queries improved from `hey headache dizziness two` / `based what` to
  `headache dizziness khat blood pressure` / `headache dizziness khat migraine blood`.
  `buildCaseQuery(caseText)` survives only as a last resort when the reply yields nothing.
- **Deep search inside each cited reference** (`refs_deep` button, auto-run on open via
  `deepRefRunAll`): for every cited reference `deepRefFetch` runs a **site-scoped** search — Brave
  `search.brave.com/search?q=site:<domain> <caseQuery>` (honours `site:`, parsed by
  `parseSerpSnippets` reading `div.snippet` → `.title` + `.content`) and, if that yields nothing and
  the ref has a `q` template, fetches + parses the reference's own search page (`parseGenericResults`).
  `refCleanItems` then drops nav/junk titles (`refJunkTitle`: "Accessibility help", cookie/privacy
  links, site homepages, titles identical to the domain) and promotes the items whose title/snippet
  actually contain a query token (`refScoreItem`); in the generic (own-site search page) fallback it
  keeps *only* query-matching titles, otherwise the scraper returns nav junk. If no surviving item
  matches the long topic at all, `deepRefFetch` retries once with the `shortTopicQuery` term
  (`deepRefFetchOnce` does the fetch).
  **`deepRefRunAll` runs the references SEQUENTIALLY (450 ms apart)** — with the old 3 parallel
  workers Brave returned empty SERPs for most refs (rate limiting), which silently pushed every card
  into the junk-prone generic fallback. Sequential + delay gives reliable, on-topic results for all
  cited refs; all `deepRes_<i>` boxes are pre-filled with the loading state so the queue is visible.
  Results render as `.liveItem` links inside the card under `نتائج الحالة داخل المرجع` so the case
  content is retrieved from *within* the source's domain — not just a link. `cleanTitle`/`cleanSnippet`
  strip the engine's site-name/breadcrumb prefixes and dates. `MENTIONED` holds the current
  `{name, ref}` list (index = `deepRes_<i>` container).
- `fetchTranslate(t, sl)` / `guessSrcLang(t)` — EN translation for queries. `translate.googleapis.com`
  is unreachable through `super-fetch-plugin` from the page ("Failed to fetch" — re-verified), so the
  chain is **`clients5.google.com/translate_a/t?client=dict-chrome-ex&sl=<src||auto>&tl=en&q=…`**
  (JSON flat array of strings; up to ~1000 chars per request — the old 180/240-char slice is gone)
  → `api.mymemory.translated.net` (only when the source language is known, langpair `<src>|en`,
  ≤180 chars) as fallback. Persian (`پچژگ`) is detected as `fa`. Always returns `""` on failure —
  callers keep the raw text.
- `trText(text, to, limit)` / `trBatch(texts, to)` — general multi-language translation (drugs search,
  UI language switching). Same `clients5.google.com/translate_a/t` endpoint (verified for `tl=en`,
  `fr`, …; it is the *only* working Google translate host through super-fetch), chunked with
  `smartChunks(text, 900)` (sentence-aware, ~900 chars/request — bigger requests fail) and rejoined;
  the parser accepts a bare JSON string, a flat array of strings (filters 2-3-letter lang codes) or
  an array-of-arrays ("`[[text, src], …]`"). Memoised in `_trCache`. Returns the input unchanged on
  failure. `trBatch` joins the pieces with a `#§#` separator for a single request and re-splits.
- `buildCaseQuery(txt)` — English search query from arbitrary case/user text: Arabic-heavy text is
  translated via `fetchTranslate`, then `refQueryFromLatin` strips stopwords and keeps up to 16 tokens
  (e.g. `25-year-old severe pain right lower quadrant abdomen nausea vomiting fever two`).
- `matchRef(name)` maps a cited name to a `REFS` catalog entry. `renderRefs()` (called from
  `showTab("refs")`) deliberately does NOT clear `#refsMentioned`, so the panel persists until the
  user hits its ✕ إزالة button.

Note: the deep search issues one Brave request per cited reference (sequential, 450 ms apart) plus the
`liveRefs` API calls, so opening the panel fires several requests — expected for the deep behaviour.

Every AI answer is instructed (OUTPUT RULES in `systemPrompt`) to cite connected references, to end
with the hidden machine-readable `🔍 TERMS: …` line (used by the 🔗 button — see `replyTermsMeta`
above; it is stripped from the displayed/stored/saved text), and to close with a `🔎 المراجع:` line.
Chat/assessment/file bubbles also get action buttons: 📋 نسخ, 🔗 فتح المرجع
(`openRefsFromReply`) and, **next to it**, 📥 **رفع إلى الملف** (`chat_save_file`) which appends the
user's message + the doctor's reply to the active patient file (`patLog`) — the file-save action on
the image/file bubbles is the existing 💾 حفظ في ملف الحالة.

## Directory (dirTab) — global + Yemen

The Directory tab is **global by default selection**: a **country `<select>` (`#dirCountry`) sits above
the search box** inside `#dirTab`. `YDIR.country` (default `"YE"`) drives it, `dirSetCountry(iso)`
changes it, and `dirCountryFill()` keeps the option list in sync (guarded by `_dirCFillKey` so the
per-keystroke `renderDir()` doesn't rebuild 188 options every time).

- **Yemen (`YE`)** — keeps the *entire* original YemenMD guide (all 12 sub-tabs) described below.
- **Any other country** — `renderDir()` calls `renderWorld(list, done)` which streams
  `src/WorldHospitals/hospitals.csv.gz` through `whText()` (fetch + `DecompressionStream("gzip")`) and
  `parseCSV`. Cards (`worldCard`) show the name (Arabic when available, else English) + the alt name,
  a Google-Maps directions button, and the website when present; search matches **both** the English
  and Arabic names. With an active health file that has GPS the list is **sorted by distance**
  (`haversineKm`) and each card shows `📍 الأقرب — x كم`; otherwise it's sorted by localized name.
  Results page 60 at a time (`yRender` + "عرض المزيد").
- `COUNTRIES` — 188 `[iso, en, ar]` triples (Yemen first, the rest sorted by localized name via
  `countryName(iso, arabic)`); helpers `countryLabel`, `flagEmoji`, `isoFromName`, `COUNTRY_BY_ISO`.
- `#dirNote` (`dirNote`) shows the global-directory hint in world mode and the Yemen hint in YE mode;
  the `sub` tabs row is hidden outside Yemen.

### `src/WorldHospitals/hospitals.csv.gz` — rebuild recipe

22 008 medical facilities, 188 countries, ~11 342 with a website. Columns:
`iso,name,ar,lat,lon,site` (values are unquoted unless a name contains a comma; `parseCSV` handles
that). Built from **Wikidata** via the **QLever** SPARQL endpoint
(`https://qlever.cs.uni-freiburg.de/api/wikidata`): query items that are instances of
hospital (`Q16917`) / clinic (`Q1774898`) / medical centre (`Q1059324`) with `wdt:P17` (country),
`wdt:P625` (coords) and optionally `wdt:P856` (website) and an Arabic `rdfs:label`. The three
country / arabic-label / website lookups were fetched in batches and joined, then gzip-compressed.
To rebuild: run the SPARQL batches, dump JSON to `scratch/`, join + emit CSV in `execute_js`, gzip
with `CompressionStream("gzip")` and write to `src/WorldHospitals/hospitals.csv.gz`.

### Yemen datasets (`src/YemenMd_tables/`)

Yemen data is fetched at runtime with `fetch("src/YemenMd_tables/<file>.csv")` and parsed by `parseCSV`
(a small RFC-4180-ish parser handling quoted fields, embedded commas and newlines) inside `yLoad(file)`,
which caches each file's parsed rows (`_yemCache`, promise-keyed so parallel calls share one fetch).

**Compact rewritten datasets (`.csv.gz`).** The original raw export included very large files
(`commercial_medicines.csv` 18.3 MB / 12 330 rows, `terms.csv` 3.5 MB, `scientific_medicines_precautions.csv`
1.3 MB) whose bulk was useless to the app: `commercial_medicines.package_insert` alone was 12 MB spread
over just 86 rows, and the CSVs carried `id`/`created_at`/`updated_at` noise. Rather than ship the raw
dumps (a multi-tens-of-MB `src/` tree could not be saved/published and was heavy on mobile), that data was
**extracted, deduped, trimmed and re-sorted** into three compact gzipped CSVs (~1.9 MB total, from ~5 MB raw):

| File | Rows | Columns | Notes |
|------|------|---------|-------|
| `commercial_medicines.csv.gz` | 11 426 | `arabic_name,english_name,drug_form,manufacturer,info` | deduped by name+form; `package_insert` + id/timestamps dropped; manufacturer name joined from `drug_manufacturers.csv`; sorted by Arabic name |
| `terms.csv.gz` | 5 785 | `arabic_name,english_name,description` | medical glossary; deduped by ar+en; sorted by Arabic name |
| `scientific_medicines_precautions.csv.gz` | 5 814 | `medicine_id,precaution` | joined to `scientific_medicines.csv` by `medicine_id` |

`.gz` files are decompressed in the browser: `yText(file)` fetches and, for a `.gz` path, pipes the
response body through `DecompressionStream("gzip")`; the text then goes through the same `parseCSV`.
To edit this data: gunzip → edit → re-gzip (`CompressionStream("gzip")`). The raw source dumps live in
the YemenMD database, not in this repo. If a future change re-adds large UNcompressed files, verify the
generator still saves.

Files kept (all ≤ ~750 KB) and their sub-tab mapping:
| Sub-tab | key | Source CSV(s) |
|---------|-----|---------------|
| 🏥 منشآت صحية | places | `establishments.csv` (+ governorates, specialties) |
| 👨‍⚕️ أطباء | doctors | `crews.csv` (+ establishments, specialties, governorates) |
| 🏢 شركات أدوية | companies | `companies.csv` (+ governorates) |
| 🏫 جهات مساندة | support | `support_establishments.csv` (+ governorates) |
| 💊 أدوية علمية | sci | `scientific_medicines.csv` + `drug_categories.csv` + `scientific_medicines_usages.csv` + `scientific_medicines_side_effects.csv` + `scientific_medicines_contraindications.csv` + `scientific_medicines_precautions.csv.gz` (all joined by `medicine_id`) |
| 🧴 أدوية تجارية | commercial | `commercial_medicines.csv.gz` (trade-name drug browser) |
| 📖 قاموس طبي | terms | `terms.csv.gz` (medical glossary, ar↔en) |
| 📰 أخبار | news | `news.csv` |
| 📄 مقالات | articles | `articles.csv` |
| 🎓 أبحاث | research | `researches.csv` |
| 🎬 فيديوهات | videos | `videos.csv` |
| 📢 رعاة وإعلانات | sponsors | `images.csv` (photos) + embedded `YEM_ADS` (see below) |

Other files present: `sliders.csv` + `main_ads.csv` (the sponsor/banner source data that `YEM_ADS`
was reconstructed from), `android_metadata.csv` (Android locale marker, `locale=ar_YE`).
`images.csv` now feeds the 📢 sponsors sub-tab. The YemenMD files `ads.csv` and `media.csv` arrived
as **unreadable 12-byte binary blobs** (not UTF-8, identical bytes, no recoverable content), so they
were **deleted**; their content was rebuilt inside the app instead (see "Sponsors & ads" below).

Engine pieces (all in `index.html`, `/* ---------- دليل اليمن ---------- */`):
- `YEM_TABS` — the 12 sub-tabs `[key, emoji, i18nKey]`; `YDIR` holds `{tab,q,gov,limit,country}`.
  `yemSetTab(k)` switches sub-tab (resets `YDIR.gov` and `dirType`).
- `renderDir()` — async dispatcher guarded by `_yemSeq` so a slow tab's late result can't overwrite
  a newer render. Each sub-tab has its own `yemXxx(list,fr,gs,done)` renderer.
- `yChips()` builds type-filter chips (places/support only); `yGovSel()` builds the governorate
  `<select>` for tabs that support it (`_yemGovRows` set by the renderer; hidden otherwise).
- `yRender()` slices to `YDIR.limit`, joins card HTML, wires `[data-det]` toggles and appends an
  "عرض المزيد" (load-more) button when there are more rows.
- Detail panels use `yDet(body)` → a hidden `.det` div toggled by `yWire()`.

Notes: `establishments.type` 1=center 2=clinic 3=hospital 4=polyclinic 5=lab 6=pharmacy;
`support_establishments.type` 1=company 2=education 3=insurance. `info`/`body` are trusted static
content injected as-is (other fields are `esc()`-escaped). Search (`YDIR.q`) spans the current
sub-tab's searchable fields; the query box is shared across sub-tabs (switching keeps it).

### Sponsors & ads (📢 رعاة وإعلانات)

The YemenMD export shipped `ads.csv` and `media.csv`, but both arrived as unreadable 12-byte binary
blobs, so their original rows could not be recovered. Instead the ads/media idea was **rebuilt inside
the app**: `YEM_ADS` (an embedded array in `index.html`, right after `YEM_TABS`) holds the sponsors
reconstructed from the readable `main_ads.csv` + `sliders.csv` + `images.csv` (titles/links only —
no images or videos are shipped). `yemSponsors()` renders the 📢 sub-tab: an "الإعلانات والرعاة"
section of sponsor cards (each with a `زيارة الموقع` link where a URL is known) followed by a
"الوسائط" photo section built from `images.csv`. Adding a sponsor = append an entry to `YEM_ADS`
(`{t:title, u:url, k:kind}`); the related label keys (`yem_tab_sponsors`, `yem_ads_sponsors`,
`yem_media`, `yem_visit`, `yem_photo`) live in all 11 `L10N` language objects.

## Drugs (drugTab) — global multilingual search + banned-substance filter

A single search box accepts **any language / any spelling** and always resolves to drug names.

- **Query resolution** (`drugQueryEnglish`): Arabic (or any non-Latin) queries are translated to
  English with the free `clients5.google.com/translate_a/t?client=dict-chrome-ex` endpoint through
  `root.superFetch` (`trText` / `trBatch` / `smartChunks`; `translate.googleapis.com` is unreachable
  through super-fetch, so it must not be used). The resolved English name is shown back to
  the user (`drug_resolved`: `🌐 باراسيتامول → Paracetamol`).
- **Names & class** (`rxNormLookup`): RxNorm `drugs.json` (exact + `approximateTerm`) → rxcui →
  `properties` (canonical **generic** name) + `rxClassOf` (pharmacologic class). Brand ↔ generic names
  and related names (`drug_brand`/`drug_generic`/`drug_class`/`drug_related`).
- **Clinical sections** (`fdaLabel`): openFDA/DailyMed label for the **quoted** generic/brand name
  only (no loose fallback — a loose match returns the wrong label). Sections: uses, dosage,
  interactions, contraindications, warnings, adverse reactions, pregnancy, pediatric, geriatric,
  overdose (`drug_interactions`/`drug_adverse`/`drug_preg`/`drug_pedia`/`drug_geria`/`drug_over`).
  Bodies are translated to the user's language. Sources are listed (`drug_sources`).
- **Banned-substance engine** (`isBannedDrug(normDrug)`): `DRUG_BLOCK_EN` + `DRUG_BLOCK_AR` keyword
  lists (narcotics, opioids, sedatives/hypnotics, anaesthetics, illicit/abuse, high-harm) compiled
  into ONE regex (`_banRe`) with a memo Map (`_banMemo`). It is applied to the query **and to every
  returned brand/generic name**; a banned hit renders only a red `.drugban` card (`drug_banned*`)
  and **nothing else is shown or recommended**. `#drugBanNote` states the policy above the results.
  The Yemen drug lists (`yemSci`, `yemMeds`) also filter banned names.
  Never pass URLs or long strings to `isBannedDrug` — it takes short drug names only.

## Voice (TTS)

`VOICE_ENGINE` defaults to `edge` (Microsoft Edge neural voices over `wss://speech.platform.bing.com`,
see `edgeSynthesize`); fallback is the browser `speechSynthesis` (`speakSys`). Do **not** reintroduce
Google Translate TTS.

## i18n

11 languages: ar (default), en, fr, es, de, tr, id, ur, hi, zh, fa. All UI strings live in `L10N`
objects keyed by language; `t(key)` / `setLang(l)` / `data-i="key"` attributes drive the UI.
When adding a string, add it to ALL 11 language objects.
**Known gap (ar + en only, English fallback via `t()` for the other 9):** the global-directory and
drug strings (`dir_country`, `dir_world_*`, `dir_near`, `dir_km`, `dir_website`, `drug_*`), the chat
file button (`chat_save_file`), the residence/GPS/contacts strings (`pf_f_residence`, `pf_res_hint`,
`pf_country*`, `pf_city`, `pf_district`, `pf_gps*`, `pf_f_contacts`, `pf_contacts_hint`,
`pf_add_contact`, `pf_c_name`, `pf_c_relation`, `pf_c_phone`, `nearby_title`) and the per-field upload
strings (`pf_up_image`, `pf_up_pdf`, `pf_read_*`) are currently only in `ar` and `en`. Localize them in
the other 9 before those languages are advertised as complete.
RTL languages are `ar`, `ur` and `fa` (the 4th `LANGS` field, e.g. `["fa","فارسی","🇮🇷","rtl"]`, drives
`dir="rtl"`). Per-language touch points beyond the `L10N` dicts: `BCP` (BCP-47 locale, e.g. `fa:"fa-IR"`
— used for dates/`Intl`), `EDGE_VOICES`/`EDGE_LANGS` (TTS, e.g. `fa:"fa-IR-DilaraNeural"`). `ar()` is a
language-agnostic helper (returns the Arabic string for ar/ur, English otherwise); it was deliberately
left unchanged for `fa`, so Persian uses the English MCQ/reference content rather than Arabic.

Arabic-first default: `detectLang()` returns the browser language only if it is a supported
NON-English locale, otherwise `"ar"` — so an English/unknown browser still lands in Arabic.
A first-time visitor (no saved `omni_lang`) sees the language screen; `enterApp(l)` saves the
choice. Switching language at runtime goes through `setLang(l)`, which calls `applyL10n()`
(all `[data-i]` labels + placeholders) AND `relocalizeUI()` (dynamic content that is NOT
`data-i`-driven: the memory-card greeting `#mcText`, the typing label, and the active tab's
render function — `renderMcq/renderDir/renderRefs/renderMem`). Any new dynamically-rendered
UI must be added to `relocalizeUI()` or it will stay in the previous language after a switch.
The built-in MCQ bank (`MCQ_BANK`) is Arabic-first: most items have no `op_en`, so `mcqNorm`
falls back to the Arabic fields rather than crashing.
`PAT_L10N`/`PF_L10N`/`MCQ_L10N` are merged into each `L10N[lang]` at load. `REFMSG_L10N`
(message actions + mentioned-references + file-save strings) is merged with an English fallback:
`L10N[l] = {...L10N.en, ...L10N[l], ...REFMSG_L10N[l]}` for every language. The Office-export
strings live in `DOC_L10N` (ar + en) and are merged the same way (see the Office export section).

## MCQ — endless AI rounds / اختبر نفسك

`mcqTab` is an **open-ended** test, not a fixed 15-question bank:

- Each **round** holds `MCQ_ROUND_SIZE` (15) questions. When the last one is answered the score
  screen (`renderMcqResult`, `#mcqResultPanel`) appears: circular % ring, score `X من 15`, a bar,
  lifetime totals, and a **review of every wrong answer** (correct option + explanation).
- **Rounds are always instant**: `mcqBuildRound()` takes questions from the AI pool first, then
  fills any shortfall from the built-in `MCQ_BANK` (marked in `omni_mcq_bankused` so bank items are
  not reused until exhausted). So round 1 is effectively the bank while the AI warms up.
- **The AI invents new questions continuously** in the background: `mcqRefillPool()` calls
  `root.generateText` (prompt = `MCQ_SYS`, a strict JSON schema) in batches of `MCQ_BATCH_REQ`
  until the pool holds `MCQ_POOL_TARGET` (30) unused questions. It (re)starts on tab show, on
  round start, and after every answer; a `mcqGenBusy` guard prevents overlapping runs.
- **No repeats**: every generated question carries a unique `topic` slug. Seen topics live in
  `omni_mcq_stats.seen`, are sent to the model as an ALREADY-ASKED list, and `mcqAcceptPool` also
  drops client-side duplicates. `mcqExtractArray` salvages complete objects from a response that
  was truncated mid-array (the model caps output at roughly 2 questions per call).
- Notes: the pool is per-language (`lg`), so switching language accumulates fresh questions for
  that language; AI-created questions are single-language (unlike the bilingual `MCQ_BANK`).
  Categories: `mcqTypeLabel` shows the localized `mcq_t_<tag>` label for the 8 bank tags, else
  prints the model's category label verbatim. Strings are `mcq_*` keys in `MCQ_L10N` /
  `MCQ_READY_L10N`, merged into every `L10N[lang]` at load.

## Health file / الملف الصحي (patient profile)

A full per-person health-record builder, opened from **Settings → 🩺 ملفي الصحي**
(`openHealthFiles()` hides the settings modal and shows `#profileScreen.on`; `closeHealthFiles()`
hides it). All UI text is `pf_*` keys in `PF_L10N`, merged into every `L10N[lang]` object at load —
add new strings to ALL 11 languages there, not only Arabic.

- Views (`pfViewName` = `list` | `form` | `review`): `pfShowList` (saved files + `ابدأ` start
  button), `pfFormHtml`/`pfWireForm`/`pfShowForm` (the form), `pfShowReview` (review page with
  **Edit / Save / Update later** — `pfEditBtn`/`pfSaveBtn`/`pfLaterBtn`). `pfRerender()` re-renders
  whichever view is open (called from `relocalizeUI` on a language switch).
- Fields: name (required), contact, **🏠 residence** (country `<select>` of 188 via `pfCountryOptions`,
  city, district/area) with a **📍 GPS** button (`pfGetGps` — `navigator.geolocation`, then
  BigDataCloud `reverse-geocode-client` through `superFetch` to auto-fill country/city/district),
  **☎️ emergency contacts** (`pfContacts`/`pfRenderContacts` — add/remove rows of name/relation/phone),
  DOB **or** age, sex (male/female/prefer-not-to-disclose),
  optional ancestry (`PF_ANC`, 10 options incl. self-described & undisclosed), optional height &
  weight with unit chips + auto-conversion (`pfCompute`/`pfSyncHw`/`pfSetWeightUnit`), then 8 free
  textareas (conditions, meds, allergies, surgeries, family, lifestyle, tests, other).
- **Per-field upload (image / PDF → auto-fill)**: each of the 8 textarea cards is built by
  `pfAreaCard(icon, field)` and carries two small icon buttons in its `h3` (🖼️ image, 📄 PDF,
  class `.pfUp`, `data-up`=field). They call `pfPickUpload(field,kind)` → hidden
  `#pfUploadInput` (`multiple`) → `pfHandleUpload()`, which loops over the picked files, appending
  each result to the textarea. `pfExtractField(field,file)` routes images through
  `fileToB64` + `askAI(...,{mime,b64})` (vision `generateText`) and PDFs/DOCX/text through
  `extractPdfText`/`extractDocxText`/`readMaybeText`, then asks the model to return ONLY the
  lines belonging in that field (keyed by `pf_f_<field>` label + `pf_<field>_ph` guidance);
  output `NONE` = nothing found. The result is **appended** to the textarea (manual text is kept),
  `pfDraft[field]` is synced, and the card shows a live `.pfUpNote` (busy spinner → ok/warn).
  Manual typing still works (`pfWireForm` `[data-pf]` oninput). New keys `pf_up_image`/`pf_up_pdf`/
  `pf_read_reading|_done|_none|_fail` are currently added to **ar + en only** — other languages fall
  back to the English string via `t()`; add them to the other 9 `PF_L10N` blocks when localizing.
- DOB/age: `pfDraft.dobMode` = `dob` (day/month/year → `pfAgeFromDob`) or `age` (y/m/w/d, supports
  newborns). `pfAgeText` renders the computed age; height supports cm or ft/in (`pfFtIn`), weight
  kg or lb (auto-converts on unit switch).
- Storage: `localStorage` keys `omni_healthfiles` (array of records `{id,name,computed,updated,draft,...}`)
  and `omni_healthfile_active` (id of the active file). Helpers `pfAll`/`pfSaveAll`/`pfActive`.
- The active file feeds the AI as the patient's case: `profileText()` returns
  `pfHealthText(pfActive())` when a health file is active (falling back to the old `omni_profile`),
  and it is passed as `prof` in both the chat path and `finishFlow` (`TASK_ASSESS`).
- `pfHealthText` also emits the **Residence (country, city, area)** line, the **Residence GPS** line
  (with a maps link) and **Emergency contacts**, so the AI can reason about location; `pfDraft` keys
  are stored flat on the record (`f.resCountry`, `f.resCity`, `f.resDistrict`, `f.lat`, `f.lon`,
  `f.contacts`), and `pfSaveFile` re-normalises `rec.contacts = pfContacts()`.

## Patient files / ملفات المرضى (doctor memory tab)

The **ذاكرة الطبيب** (`memTab`) tab now stores every conversation and case as a **file tab named
after its owner**: open a file and you see the whole transcript (user messages + AI replies) with
the day header and the time of each message, plus that patient's saved case reports.

- UI (`renderMem` → `renderPatTabs` → `renderPatFile`): `#patTabs` = one rounded `.patTab` per
  patient (name + message count + 🟢 when it is the active file); `#patFileBody` = the opened file
  card (name, created/last-updated dates, action buttons, then `الرسائل والردود` and
  `تقارير الحالات` sections). The old standalone "saved cases" panel was folded into these files.
- Data: `localStorage` key `omni_patients` — array of `{id, name, created, updated, msgs:[{r:"u"|"a", t, d}]}`
  (`r` = role, `t` = raw text/markdown, `d` = epoch ms). `omni_active_patient` stores the id of the
  working file. Helpers: `patAll`/`patSaveAll`/`patEnsure`/`patLog`/`patCasesAll`/`patDelFile`/
  `patRename`/`patOpen`/`patClose`/`patDownload`.
- Attribution: `curPatient()` prefers `omni_active_patient`, then the active health file (`pfActive`),
  then a `PAT_GENERAL` ("ملف عام") file. So creating/activating a health file also names the patient
  file (`pfSetActive` syncs both). Logging is done at the call sites — `sendUserText`, `answerFlow`,
  `aiReply`, `finishFlow`, `handleImageFile`, `handleAnyFile` — and `saveCase` stamps `pid`/`pname`
  on each case so reports list under the right file. Cases saved before this feature (no `pid`) show
  in the general file.
- Actions per file: **تعيين كنشط** (routes all future messages/cases there), **تعديل** (inline rename,
  `patRename` — no native dialog), **تحميل Word** (whole transcript via `downloadDoc`/`downloadWordDoc`),
  **حذف الملف**, **إغلاق**. Strings are `mem_*` keys in `PAT_L10N` (+ `mem_activeHint` in `PAT_HINT`),
  merged into every `L10N[lang]` at load — add new strings to ALL 11 languages.
- **Per-message actions** (each saved message in `renderPatFile` carries `data-msg="<pid>:<index>"`):
  🔽/🔼 **طي/توسيع** (`patToggleFold` — persists `m.fold`, renders a one-line preview when collapsed),
  ✏️ **تعديل** (`patEditMsg` — inline `<textarea>` + save/cancel, stamps `m.edited`),
  ↩️ **فتح في المحادثة** (`patReopenMsg` — sets this file active, re-adds the bubble to the chat and
  loads the text into `#chatInput` so it can be edited/resent), 🗑️ **حذف** (`patDelMsg`, with a
  confirm). Lookup is `patGetMsg(pid, idx)`. These keys are `mem_delMsg*`/`mem_edit*`/`mem_fold`/
  `mem_expand`/`mem_reopen*` in `REFMSG_L10N` (below).
- Deleting a file only removes that patient record (and its `pid`-stamped cases); health files are
  untouched. `clearMemory()` also clears `omni_patients` + `omni_active_patient`.

## Office export — Word `.docx` + Excel `.xlsx`

Everything here is **hand-built real OOXML** (no HTML-`.doc`/CSV-in-disguise): the exported files are
genuine `wordprocessingml` / `spreadsheetml` zip packages, so they open natively in Microsoft Office,
LibreOffice, WPS, Google Docs, Pages, mobile Office, etc. There is **no external dependency** — the
zip writer, the Word/Excel XML and the embedded logo are all produced in-page.

### Shared OOXML helpers (top of the export block)

- `oxZip(files)` — minimal ZIP (store/no compression) writer: builds local file headers + central
  directory + EOCD from `[{name,data}]`, with real CRC-32 (`oxCrc`). `oxSave(blob,name,mime)` creates
  an object URL, clicks a hidden `<a download>`, then revokes it; `oxStamp()` → filename timestamp;
  `oxFmtDT(ms)` → local date/time string; `xesc(s)` → XML escaping.
- `oxLogoBytes()` — fetches the brand logo URL once (`LOGO`) and caches the `Uint8Array` + its
  content-type, so the same image can be embedded into both docx headers and (potentially) xlsx.

### Word (`buildWordDoc` / `downloadWordDoc`)

`buildWordDoc(data)` is **async → Blob**; `downloadWordDoc(data)` is the thin wrapper (calls it, then
`oxSave(...,".docx",…)` + `toast(docSaved)`). `data` = `{patientName, profileLines, messages,
assessment, cases, date}`. Page setup is **A4** (`w:pgSz w=11906 h=16838` twips, margins `1134`), with
`w:bidi` for RTL. Structure, top to bottom:

- **Header** (`word/header1.xml`, `titlePg`-less so it repeats on every page): embedded logo raster
  (`word/media/logo.png` via `a:blip r:embed="rId1"`) + the brand **Dr Omni AI** + `docTag` tagline +
  a blue rule. The logo is fetched through `oxLogoBytes()`; if the fetch fails the header degrades to
  the text brand only (never a broken image relationship).
- Centered `docTitle`, `docTag`, and `docDate`; optional `docPatientName`.
- **Patient table** — `wTable` with `labelCol:true` (highlighted label column) + blue header shading.
- **Messages table** (when `messages.length`) — columns `docNo/docRole/docTime/docText`, blue header row.
- **Assessment** — `docAssessment` heading + `mwBlocks(assessment)`, a tiny markdown→OOXML renderer
  (bold, bullet/numbered lists, `#` headings, paragraphs).
- **Case-reports table** (when `cases.length`) — `docNo/docTime/docCC/docAssessment/docPearl`.
- Red emergency band (`docEmergency`), signature/stamp line (`docSignature`/`docStamp`) and disclaimer.
- **Footer** (`word/footer1.xml`): brand line, `صفحة PAGE من NUMPAGES` **field codes** (`w:fldChar` /
  `w:instrText`, so Word/Word-Online recompute the real total), red emergency line, and a
  generated-by + date + website (`docWeb`) line.

`wRun(text,{b,sz,color,rtl})` / `wRuns` / `wPara(content,{rtl,align,spacing,shd})` / `wTable(rows,
{rtl,header,labelCol,widths,zebra})` are the low-level builders; `packDocx(bodyXml,rtl,title,logo)`
assembles `[Content_Types].xml`, `_rels/.rels`, `word/document.xml`, `word/_rels/document.xml.rels`,
`word/styles.xml`, `word/header1.xml`, `word/footer1.xml`, `word/media/logo.png` and the core/app props.

### Excel (`buildCasesExcel` / `downloadCasesExcel`)

`buildCasesExcel(pid)` is **sync → Blob**; `downloadCasesExcel(pid)` wraps it with
`oxSave(...,".xlsx",…)` + `toast(xlSaved)`. It auto-tabulates the saved cases:

- Sheet 1 `xl_sheet1` (**الحالات** / Cases) — one row per saved case, columns `xl_no / xl_date /
  xl_patient / xl_cc / xl_ptdata / xl_assess / xl_pearl` (emergency data, full assessment, lesson).
- Sheet 2 `xl_sheet2` (**ملفات المرضى** / Patient files) — one row per patient file, columns `xl_no /
  xl_name / xl_created / xl_updated / xl_msgs / xl_cases`.
- `xSheet`/`packXlsx` build styled `spreadsheetml`: blue (`FF2A6FD6`) header row (white bold) with
  `autoFilter`, thin `FFC8D9FA` borders, `wrapText` cells, RTL (`rightToLeft="1"` when the UI is RTL),
  and fixed column widths.

### Where it is triggered

- `memTab` first panel: a **📊 تحميل Excel لكل الحالات** `bigBtn` → `downloadCasesExcel()` (all cases).
- Per patient file (`renderPatFile` action row): **📊 Excel** (`data-patact="xl"` →
  `downloadCasesExcel(p.id)`) and **📄 Word** (`patDownload` → whole transcript `downloadWordDoc`).
- After an assessment (`finishFlow` result actions): **📊** Excel for that patient and **📄** Word for
  the single case (passes `patientName` + `pearl`).
- `patDownload(p)` (server-side path) also passes `messages[]` + `cases[]` + counts.

`DOC_L10N` holds the export strings for **ar + en**; it is merged into every language with an English
fallback (`L10N[l] = {...L10N.en, ...L10N[l], ...DOC_L10N[l]}`). Keys: `dlExcel`, `xlSaved`,
`docPatientName`, `docSignature`, `docStamp`, `docMessages`, `docCases`, `docPearl`, `docCC`,
`docRole`, `docTime`, `docText`, `docNo`, `docWeb`, `docPage`, `docOf`, and the `xl_*` set
(`xl_no xl_date xl_patient xl_cc xl_ptdata xl_assess xl_pearl xl_name xl_created xl_updated xl_msgs
xl_cases xl_sheet1 xl_sheet2 xl_exportAll xl_exportOne`).

### Testing recipe

`buildWordDoc`/`buildCasesExcel` are separately callable (the download wrappers only add `oxSave` +
toast), so they can be exercised from `page_eval` and the resulting Blob read as bytes. Verified by
unzipping the output and validating every XML part, and by rendering the docx with `docx-preview` +
`vision` (header logo/brand, tables, red band and footer all placed correctly). When changing this
block, re-render and vision-check rather than trusting a clean console.

## Persistence

`localStorage` via `save`/`load` helpers (keys: `omni_cases`, `omni_pearls`, `omni_facts`,
`omni_profile`, `omni_healthfiles`, `omni_healthfile_active`, `omni_patients`,
`omni_active_patient`, `omni_mcq_stats` (rounds/total/correct + `seen` topics),
`omni_mcq_pool` (unused AI questions), `omni_mcq_bankused`, `omni_vengine`, ...).
Per-user, per-generator origin.

## Conventions

- ids end with their type: `...El`, `...Btn`, `...Ctn`, `...Input`.
- Prefer `hidden` attribute over `display:none`.
- Always show an animated loading indicator for AI/async work.
- Test responsive layouts (phone 390×844 and desktop) and check `vision` for visual regressions.
