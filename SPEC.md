# Fashion Citywalk — product spec (brief)

- **Product:** Personalized Fashion Citywalk Paper-Cutout Animation.
- **Price / channel:** $12.99 on Gumroad —
  https://binbinster.gumroad.com/l/okgucl
  (product_id `41J22OJ3Jnh_odM50cMePg==`, license keys ON).
- **Storefront:** https://api.newkwlab.com/walk/ (walk-web service,
  loopback port 8796, Caddy `handle /walk*`).
- **Format:** one 10-second vertical MP4 (1080×1920) per order,
  pop-groove soundtrack included; delivered by email within 48 hours
  plus a token download link on the order page.

## Order shape

- Solo only — exactly ONE photo per order (JPG/PNG ≤ 10 MB). No duo.
- **Template is a required field**, one of:
  `fashion` (01 Fashion Citywalk), `paris` (02 Paris Citywalk),
  `newyork` (03 New York Citywalk), `tokyo` (04 Tokyo Citywalk).
  It is stored on the order record and exposed in the order-status
  response and in `GET /walk/agent/paid` (`template` +
  `template_label`).
- Optional: first name, free-text note. Required: buyer email,
  license key.
- One Gumroad license key = one film (verified fail-closed against
  Gumroad at activation; only last-4 stored).

## Flow

Same as PaperStage (/clip): create order → activate with license key
→ producer builds the film by hand in the chosen template →
`POST /walk/agent/deliver/<id>` → buyer downloads at
`/walk/files/<id>/<delivery_token>/film.mp4`.

## Architecture

Clone of `~/workspace/paperstage-site/` with `/clip` → `/walk`,
`clip_config.json` → `fashion_config.json`, `CLIP_PORT` → `WALK_PORT`
(default 8796), scenes/variants replaced by the four templates and
the single-photo rule. Stdlib-only Python; see BUILD-REPORT.md for
what was built and deploy/README.md for the (not yet performed)
deployment.
