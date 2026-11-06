# Six-Template Swap — Build Report (LOCAL ONLY, not deployed)

Date: 2026-11-07. Scope: bring the Fashion Citywalk site from two
templates (fashion, tokyo) to the final six-template set. Built and
smoke-tested locally in `~/workspace/fashionwalk-site/`; **nothing was
deployed** — the live EC2 copy still serves the two-template version
until a separate deploy task runs.

Final set:
01 fashion — Fashion Citywalk · 02 tokyo — Tokyo Citywalk ·
03 cafe — Shanghai Citywalk (Café) · 04 collage — Off Duty Collage ·
05 duo — London Citywalk Duo · 06 market — Hong Kong Citywalk (Market)

## Changed files

| File | Before | After | Change |
|---|---|---|---|
| `web/site/index.html` | 54,524 B | 64,486 B | 4 new poster sections, 4 new picker cards, count copy → six, 1 CSS class added |
| `web/app.py` | 20,362 B | 20,456 B | `TEMPLATES` whitelist only (see below) |
| `web/site/llms.txt` | 4,177 B | 5,086 B | Template list 2 → 6 with descriptions; order-step values; facts line |

New media in `web/site/media/` (verbatim copies of the v3 finals):

| File | Bytes | Source |
|---|---|---|
| `cafe-citywalk.mp4` | 4,705,215 | `paper-cutout-fashion-v3/ta-cafe/cafe-citywalk.mp4` |
| `cafe-citywalk-poster.jpg` | 359,504 | frame @ 8.8 s (standing, coffee in hand) |
| `offduty-collage.mp4` | 4,007,139 | `paper-cutout-fashion-v3/tb-collage/offduty-collage.mp4` |
| `offduty-collage-poster.jpg` | 350,704 | frame @ 2.2 s (full magazine page) |
| `london-crossing-duo.mp4` | 5,122,901 | `paper-cutout-fashion-v3/tc-duo/london-crossing-duo.mp4` |
| `london-crossing-duo-poster.jpg` | 275,322 | frame @ 5.4 s (big stride + small walker) |
| `hongkong-market.mp4` | 3,973,551 | `paper-cutout-fashion-v3/td-market/hongkong-market.mp4` (post-repair) |
| `hongkong-market-poster.jpg` | 327,844 | frame @ 8.8 s (standing at stall, paper bag) |

All four posters were visually inspected after extraction — each is a
clean standing/hero frame with the film's letter-tile title legible.

## index.html details

- Four `<article class="poster …">` sections inserted after the Tokyo
  section, same design system (strip + awning/asphalt band + polaroid
  video + letter-tile h3 + idxcard + fluoro label + p-foot):
  - 03 Café — `reg-cafe` (browns/greens) + awning band; tiles
    "Shanghai / Citywalk"; fluoro "Coffee o'clock."
  - 04 Collage — new `reg-journal` class (newsprint register,
    `#ece4cf`) + ink/cream band, flipped grid; tiles "OFF / DUTY";
    fluoro "Off duty."
  - 05 Duo — `reg-crossing` + asphalt band; tiles "London /
    Citywalk"; fluoro "Two ways to cross."
  - 06 Market — `reg-market` + red/white awning band, flipped grid;
    tiles "Hong Kong / Citywalk"; fluoro "Fresh today."
- Order-form picker: four new `.vcard` cards (`data-tpl` cafe /
  collage / duo / market) with poster thumbnails, numbered 03–06.
  Existing picker JS binds all `#tplGrid .vcard` and all
  `[data-tpl].btn` automatically — no JS changes were needed.
- Count copy updated everywhere: top strip "Vol. 01 · Six Cities",
  hero eyebrow "One photo · Six looks", line-up head "Six" + "Six
  posters · Six moods" + "Six templates, six moods", spec card
  "Six paper looks" (lists all six), meta description (lists all
  six), footer product line ("six looks, from Fashion to the Hong
  Kong market"). Renumbering: fashion 01, tokyo 02 (already),
  cafe 03, collage 04, duo 05, market 06 — strip labels, idxcards,
  figcaptions and picker titles all agree.
- No stale copy remains: greps for "Four templates", "Four paper
  cities", "Fashion or Tokyo", "Two templates" return zero hits
  ("Four steps, three minutes" in the process section refers to the
  order steps and was correctly left alone).

## app.py details

Only the `TEMPLATES` dict changed — now exactly:

```python
TEMPLATES = {
    "fashion": "01 Fashion Citywalk",
    "tokyo": "02 Tokyo Citywalk",
    "cafe": "03 Shanghai Citywalk (Café)",
    "collage": "04 Off Duty Collage",
    "duo": "05 London Citywalk Duo",
    "market": "06 Hong Kong Citywalk (Market)",
}
```

`paris` / `newyork` are gone from the whitelist. Ports, endpoints,
config keys and the license flow are untouched (verified by diff
scope: the edit touched the dict only). `ast.parse` passes.

## llms.txt details

- Template list now lists all six values with one-line descriptions.
- Ordering step 2 names all six values as the required field.
- One judgment call: the old facts line said "There is no duo
  option — every Citywalk is a solo walk", which contradicts the new
  Duo template. It now reads: "One template per film; the six
  templates are the ones listed above. Every film is made from ONE
  photo of ONE person — the Duo template included: it stages that
  one person at two scales in a single crossing scene." All other
  facts (price, format, delivery, safety lines) are byte-identical.

## Smoke test (local, scratch copy on port 8899)

Ran the app from a scratch copy (`/tmp/walk-smoke`, real
`fashion_config.json`) so no test orders touched the real `data/`
dir; scratch deleted and the test process killed afterwards
(port 8899 confirmed free).

- `GET /walk/health` → 200 `{"ok": true}`
- `GET /walk/` → 200, 64,486 B; page contains all six template
  names (Fashion ×9, Tokyo ×5, Shanghai ×5, Off Duty ×7, London ×5,
  Hong Kong ×5 mentions).
- All 12 media files (6 films + 6 posters) → HTTP 200 with byte
  sizes matching the files on disk.
- `GET /walk/llms.txt` → lists all six template values.
- `POST /walk/api/orders` with `template=paris` → **400
  `bad_template`**; `template=newyork` → **400** as well.
- `POST /walk/api/orders` with each of fashion / tokyo / cafe /
  collage / duo / market → **200** `{"status": "received"}`.
- Café order `WPvJlgB0vjExNDyV`: status endpoint reports
  `template: "cafe"`, `template_label: "03 Shanghai Citywalk
  (Café)"`, status `received`.
- Activate that order with a fake license key → **403
  `license_invalid`** after a real round-trip to Gumroad's verify
  API — fail-closed, order stays `received`, exactly the pre-swap
  behaviour. Empty key → 400 `missing_license_key`.

## Notes for whoever deploys

- The cut `paris-citywalk.*` / `newyork-citywalk.*` media files are
  still in `web/site/media/` (unreferenced by the page). Harmless;
  delete them in the deploy pass if a clean media dir is wanted.
- Deploy needs: new `index.html`, `llms.txt`, `app.py` (requires a
  `walk-web.service` restart — the whitelist lives in code) and the
  eight new media files.
