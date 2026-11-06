# Citywalk /walk — Poster-Board Redesign

Front-end-only redesign of `web/site/index.html`. `web/app.py`, config, API flow,
form field names/ids and the page's `<script>` logic are untouched (script carried
over byte-identical — see Verification). Not deployed.

## Why

The old page wore the house editorial skin (warm paper + Poppins / Source Serif 4 /
IBM Plex Mono) shared with the duo/book/clip sites. The user rejected that: the
product is a multi-style paper-collage fashion line, so the site must look like
the product — a Cooper-style poster board — not like our other sites.

## Examples studied (motionsites-web-builder, step 2)

Picked from `examples/INDEX.md` by closeness to "fashion + city posters + single-file":

1. **prompt-hero.md** (PROMPT, fashion/archive landing) — fashion register;
   scattered, deliberately un-gridded gallery layout; entry choreography with
   written-out stagger delays (logo 0s / nav .15s / caption .3s / info .45s);
   mono-tight metadata at 12–15px; restraint: the media carries everything.
   Borrowed: stagger discipline, scattered-board composition, metadata scale.
2. **neo-museum.md** (Neo Museum) — exhibition system: `[ 02 ] Section` mono
   labels at `tracking .2em`, chapter indices `01 / 05`, off-white `#fcfcfc`
   paper ground, hairline rules, hard-shadow hover (`3px 3px 0`). Borrowed:
   the info-strip / index system that becomes our four-zone poster headers.
3. **mostar-guide.md** (Mostar city guide) — single standalone vanilla HTML
   file for a *city* product; paper `#fdf1e1` + ink `#111411` tokens; tag pills
   (min-height 42px, paper fill, soft shadow); serif display numerals in a
   facts grid; explicit per-breakpoint value tables. Borrowed: paper/ink
   token discipline, pill/tag sizing, city-guide tone.

## Spec (values decided before code)

### Concept
The page is a corkboard of Cooper-style posters. Signature: ransom-note
letter-tile headlines (styled spans, mixed black/cream/rust/slate/fluoro tiles,
±2° rotations) + taped polaroid frames for every film. Four-zone poster
structure per section: info strip → title → scene → fluoro micro-label.

### Fonts (Google Fonts, display=swap)
- **Archivo** 400/500/600/700/800/900 — tile letters (900), headings, body, buttons.
- **Instrument Serif** italic 400 — accent words, polaroid captions, big numerals.
- **Space Mono** 400/700 — info strips, micro-labels, chips (10.5–11.5px, ls .16–.2em, uppercase).

### Colour tokens (sampled from the hero film's own poster frame)
```
--board:#eee4cb   --paper:#fffdf4   --ink:#26211a
--ink70/55/40: rgba(38,33,26,.7/.55/.4)
tile: black #292420 · cream #f6f0dc · rust #b3442c · slate #7e94a9 · fluoro #f7e600
crimson #bc2333 · asphalt #33383f · footer #262a30
Registers: fashion crimson #bc2333 / tint #f9e9e1 · tokyo pink #f2a7c3, cyan #93d5cf / tint #fdeef3
café (paris slot) brown #6f4a2f, green #2e5d43 / tint #f3ecdd · crossing (newyork slot) fluoro + red #c8102e / tint #f6f1de
off-duty (what-you-get) ink/paper + fluoro · market (order) red #c8352c + cream / tint #fdf6e3
```

### Key components (numbers)
- **Info strip**: Space Mono 10.5px, ls .2em, uppercase, padding 12px 0 10px, 2px ink rule under.
- **Tiles**: inline-flex letters, gap 5–7px; tile padding .07em .14em; radius 6–7px;
  shadow `0 2px 0 rgba(38,33,26,.35)`; rotations via nth-child (−2.2°/1.8°/−1°…) + ±2px baseline jitter.
  Hero tiles font `clamp(46px, 8.6vw, 102px)` Archivo 900; city tiles `clamp(30px, 4.6vw, 54px)`;
  heading tiles `clamp(24px, 3.6vw, 40px)`. Containers carry `aria-label`, letters `aria-hidden`.
- **Polaroid**: bg #fffdf6; padding 13px 13px 68px (chin deepened in round 2);
  radius 3px; shadow `0 2px 3px rgba(38,30,15,.18), 0 22px 38px rgba(38,30,15,.2)`;
  rotate −1.3°/+1°; caption Instrument Serif italic 16.5px, ink70, bottom 14px.
- **Tape**: 100×28px, `linear-gradient(rgba(243,231,190,.88), rgba(238,222,170,.78))`,
  rotate −4°/+5°, top −13px, shadow `0 2px 5px rgba(38,30,15,.14)`.
- **Push-pin**: 17px radial-gradient circle (#ff8a7a → #c8352c → #8e1b29), shadow `0 3px 4px rgba(0,0,0,.3)`.
- **Fluoro strip**: bg fluoro, ink text, Archivo 800 13.5px uppercase ls .05em,
  padding 11px 18px, rotate −1.6°, hard shadow `4px 4px 0 var(--ink)`.
- **Buttons** (radius 4px, Archivo 700 14.5px uppercase, padding 15px 24px):
  `.solid` fluoro + `4px 4px 0` ink shadow (hover −1px/shadow 5px, active +3px/1px);
  `.ink` ink bg, cream text; `.ghost` 1.5px ink border. Arrow span nudges 2px on hover.
- **Poster panel**: padding 30px (mobile 20px 18px 24px), radius 6px,
  shadow `0 16px 34px rgba(59,45,20,.15)`, per-register tint bg; desktop tilt ±.35°.
- **Zebra band**: 48px asphalt #33383f; stripes
  `repeating-linear-gradient(96deg, rgba(253,250,238,.94) 0 44px, transparent 44px 88px)`;
  zigzag torn clip-path top/bottom.
- **Awning band** (order poster): 20px, `repeating-linear-gradient(90deg,#c8352c 0 28px,#fdf3da 28px 56px)`,
  zigzag clip-path bottom edge.
- **Form**: inputs bg #fffefa, 1.5px solid ink85, radius 5px, padding 13px 15px;
  focus = ink border + `0 0 0 3px var(--fluoro)`. Picker `.vcard.sel` = 2px ink border +
  `5px 5px 0 var(--acc)` per-template accent + "PICKED" mono chip (::after).
  Upload box dashed 1.5px ink45. `.okbox` #eef4e2 / `.warnbox` #fbe9df, 1.5px ink border.
- **Footer**: asphalt #262a30, cream text, 16px mini-zebra on top, fluoro mono headings.
- **Motion**: hero tiles pop in with spring `cubic-bezier(.34,1.56,.64,1)`, .55s,
  delay `.15 + i*.045s` (--i per tile); kicker/sub/CTAs fadeUp .8s at .5/.65/.8s;
  polaroid settles 1s at .35s; scroll reveals reuse the old IO system (threshold .05);
  `prefers-reduced-motion` shows final states.

### Layout
Board wrap max 1180px, padding-inline 20px → 36px (≥768) → 48px (≥1200).
Hero grid ≥960px: `1.02fr .98fr`, gap 56px; polaroid max 430px.
Gallery posters ≥900px: rows `340px 1fr`.
Sections spaced 84px (64px mobile). Mobile 390px: tiles wrap per word,
polaroids full-width max 400px, panels stack, no horizontal scroll
(`overflow-x: clip` on body, tilts disabled <640px).

### Media slots (pure file replacement, unchanged names)
`/walk/media/fashion-citywalk.mp4|.jpg`, `paris-…`, `newyork-…`, `tokyo-…` —
hero + film section use the fashion slot; each gallery poster and picker
thumb uses its own slot. Paris/New York films are placeholders pending swap.

## Build & fix rounds

Screenshots via the skill's `scripts/shoot.py` (Playwright) against a local
static server mapping `/walk` → `web/site/`, plus per-section element captures.
Note: the sandbox browser cannot reach Google Fonts (proxy tunnel fails), so
final QA frames were rendered from a byte-identical copy whose only change is
the font stylesheet URL pointing at locally downloaded copies of the same
woff2 files — typography in the finals is the real Archivo / Instrument Serif /
Space Mono. `/walk/api/config` 404s under the static server; the page JS
catches it (checkout-button fallback), which is the designed behaviour.

**Round 1** (first full review — hero, all 4 posters, gets, zebra/steps, tips,
order, status, footer, mobile hero):
- Hero, tiles, posters, zebra band, awning, order desk all landed on first pass.
- Weak point 1: poster panels — copy column vertically centred against the tall
  polaroid left the panel's lower-right dead, and the four-zone structure had
  no zone 4. **Fix:** added a `.p-foot` dashed footer strip to all four posters
  ("Made to order · Checked by a person / Citywalk Studio", Space Mono 9.5px).
- Weak point 2: "What you get" card A spans two grid rows but held one short
  paragraph — large dead area. **Fix:** card A is now a flex column with a
  spec list pinned to its base (Format MP4 / Frame 1080×1920 / Length
  10 seconds / Sound Pop groove — all facts already stated on the page).

**Round 2** (geometry check with real fonts):
- Measured hero polaroid: two-line figcaption overlapped the video's bottom
  edge by 9.3px (caption height 48px + 13px offset > 52px chin). **Fix:**
  polaroid bottom padding 52px → 68px, caption 17.5px → 16.5px/1.3 line-height.
  Re-measured: 6.8px clearance, no overlap.
- Mobile order form and 390px hero re-checked: single column, PICKED tag and
  awning render, no horizontal overflow.

## Verification

- **Script/logic untouched:** both `<script>` blocks in the new file are
  byte-identical to the old file (extracted and compared programmatically;
  the main block was carried over verbatim, not retyped).
- **Form contract diff (old vs new, programmatic):** form field elements
  identical (types + ids); all old element ids present (one additive
  `id="top"` hero anchor); API endpoints identical (`/walk/api/config`,
  `/walk/api/orders`, `/walk/api/orders/`); FormData fields unchanged
  (`template`, `name`, `email`, `note`, `photo1`); `data-tpl` values unchanged
  (`fashion`, `paris`, `newyork`, `tokyo`); license/activate/status flows run
  through the same carried-over script.
- **Media:** all 8 `/walk/media/...` references resolve to existing files;
  zero broken images in the browser QA report; hero video plays
  (readyState 4, no errors); gallery videos `preload="none"` by design.
- **Layout:** no horizontal overflow at 1440px or 390px; zero page errors.
- **Weight/deps:** index.html 58,972 B (old 40,776 B — the poster design
  system is inline CSS); only external dependency is Google Fonts
  (fonts.googleapis.com / fonts.gstatic.com), as permitted.
- **Screenshots:** `redesign-shots/` — desktop.png, desktop-early.png,
  desktop-full.png, mobile.png, mobile-full.png.
- Not deployed; `web/app.py`, `web/fashion_config.json`, `llms.txt` untouched.
