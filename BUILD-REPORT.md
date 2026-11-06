# Fashion Citywalk — build report (local build, NOT deployed)

Built 2026-11-07 by cloning the PaperStage implementation
(`~/workspace/paperstage-site/`): stdlib-only Python, JSON order
store, Gumroad license verify fail-closed, agent-token production
API, hand-rolled multipart. Base path `/walk/`, port **8796**
(`WALK_PORT`), service name `walk-web`.

## Differences from the /clip pattern

- **Solo only** — the variant system is gone. Exactly one photo per
  order (server-enforced: photo1 required, no photo2 path).
- **Templates replace scenes** — `TEMPLATES = {fashion, paris,
  newyork, tokyo}` with labels "01 Fashion Citywalk" … "04 Tokyo
  Citywalk". `template` is a **required** form field (missing/invalid
  → 400 `bad_template`), stored on the order record, and exposed in
  the order-status response and in `GET /walk/agent/paid`
  (`template` + `template_label`).
- Single optional name field (`name`) instead of name1/name2.
- Config is `fashion_config.json` with the **live** Gumroad values
  (product below), a fresh `agent_token` (secrets.token_hex(24));
  delivery download filename `fashion-citywalk-film.mp4`.

## What was built

- [web/app.py](sandbox://workspace/fashionwalk-site/web/app.py) —
  server: static site + media (MP4/JPG), `POST /walk/api/orders`
  (multipart; template enum of 4 required, one photo, jpg/png
  ≤10 MB), `GET /walk/api/orders/<id>` (status incl. template),
  `POST …/activate` (fail-closed Gumroad verify, one key = one film,
  only last-4 stored), token delivery download at
  `/walk/files/<id>/<token>/film.mp4`, production API
  (`GET /walk/agent/paid` incl. template + photo URLs,
  `GET /walk/agent/orders/<id>/photo/1`,
  `POST /walk/agent/deliver/<id>` multipart `mp4` → delivered),
  health, config, media path-traversal guard.
- [web/site/index.html](sandbox://workspace/fashionwalk-site/web/site/index.html)
  — single-page storefront + order desk in the family design system
  (cream paper, Poppins / Source Serif 4 italic / IBM Plex Mono, pill
  nav) re-dressed for street fashion: crimson accent, **no snow, no
  lights strip, no Christmas anything**. 88 px serif hero ("Step onto
  a *paper* runway."), autoplay looping Fashion Citywalk film with
  poster, craft section (front walk + side stride story), **"Four
  cities" gallery with all four playable sample films** (poster +
  number + city name + "Walk in this city" button that pre-selects
  the template in the order form), what-you-get cards, 4-step
  how-it-works, photo tips + privacy (solo wording), order form
  (license panel → **visual template picker: four poster cards** →
  name → email → one photo upload → note), order-status view with
  token MP4 download.
- [web/site/llms.txt](sandbox://workspace/fashionwalk-site/web/site/llms.txt)
  — Qiaomu pattern rewritten for this product: facts ($12.99, solo,
  4 templates, 10 s format), agent steps, human-only steps (Gumroad
  payment, photo choice/permission), safety lines (key never stored
  beyond last 4, photos private and deleted after delivery, never
  invent IDs/keys/status).
- [web/fashion_config.json](sandbox://workspace/fashionwalk-site/web/fashion_config.json)
  — live values: `gumroad_product_id` =
  `41J22OJ3Jnh_odM50cMePg==`, `gumroad_buy_url` =
  https://binbinster.gumroad.com/l/okgucl , fresh `agent_token`
  generated locally (value not printed in any report).
  [fashion_config.example.json](sandbox://workspace/fashionwalk-site/fashion_config.example.json)
  at the tree root with placeholders.
- [deploy/README.md](sandbox://workspace/fashionwalk-site/deploy/README.md)
  — EC2 wiring marked **NOT yet deployed**: `/home/ubuntu/walk-web/`,
  `walk-web.service` (`WALK_PORT=8796`), Caddy `handle /walk*` →
  127.0.0.1:8796 next to `/duo*`, `/book*`, `/clip*`.
- [SPEC.md](sandbox://workspace/fashionwalk-site/SPEC.md) — brief
  product/architecture spec.

## Media (web/site/media/)

- `fashion-citywalk.mp4` — verbatim copy of
  `paper-cutout-fashion-v2/fashion-walk-v2.2.mp4` (10.0 s,
  1080×1920); `fashion-citywalk-poster.jpg` — copy of the approved
  `~/workspace/fashion-cover-v22.jpg`.
- `paris-citywalk.mp4` / `newyork-citywalk.mp4` /
  `tokyo-citywalk.mp4` — verbatim copies of the template finals
  (t2/t3/t4, each 10.0 s); posters extracted at t=9.4 s (the
  8.5–10 s final standing beat) and **visually verified** (standing
  pose, letter-tile city title, correct template number 02/03/04).

## Verification (local, loopback port 8796)

- `python3 -m py_compile web/app.py` — OK; both config JSONs parse.
- Every `/walk/media/…` reference in index.html resolves to a
  non-empty file on disk (8/8).
- Smoke run — **26/26 checks passed**: landing / llms / health /
  config (live buy URL) / all 8 media files → 200; media traversal →
  404; **unknown template → 400 `bad_template`; template omitted →
  400 (required field)**; photo omitted → 400; valid order created
  (`template: tokyo`); status response shows `template` +
  `template_label` "04 Tokyo Citywalk"; **fake license key rejected
  403 by the real Gumroad verify** ("That license does not exist for
  the provided product") — fail-closed against the live product;
  agent endpoints 403 without token, 200 with; agent photo download
  200; deliver → `delivered`; status returns `film_url`; buyer
  download 200 with delivery token, 403 with a wrong one.
- Production-API exposure of the template was additionally verified
  by flipping the smoke order to `paid` in its local order.json and
  re-querying `/walk/agent/paid`: the entry carries `template`,
  `template_label`, `name` and `photo_urls` as designed.
- Server stopped and all smoke data purged afterwards (no `data/`
  directory ships).

## Open items

1. **Not deployed** — follow deploy/README.md when the parent gives
   the word (new systemd unit `walk-web` + one Caddy handle; nothing
   existing is touched). No local process is left running.
2. The site sends no email by design (status page + producer-sent
   delivery email), same as duo/book/clip.
