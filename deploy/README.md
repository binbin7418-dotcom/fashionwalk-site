# Fashion Citywalk — deploy notes (NOT yet deployed)

Target: the same EC2 box as SnapDuo, Starlight Stories and PaperStage
(api.newkwlab.com), wired exactly like duo-web / book-web / clip-web:
a systemd service running `web/app.py` on a loopback port, with Caddy
reverse-proxying the public path prefix to it.

## Layout on EC2

```
/home/ubuntu/walk-web/
  app.py              <- from web/app.py
  fashion_config.json <- create from fashion_config.example.json (see below)
  site/               <- from web/site/ (index.html, llms.txt, media/)
  data/               <- created at runtime (orders/, licenses.json)
```

## Config

```bash
cp fashion_config.example.json fashion_config.json
# generate the agent token once and paste it in:
python3 -c "import secrets; print(secrets.token_hex(24))"
```

- `gumroad_product_id`: `41J22OJ3Jnh_odM50cMePg==` — the live
  "Personalized Fashion Citywalk Paper-Cutout Animation" product
  ($12.99, license keys enabled).
- `gumroad_buy_url`: https://binbinster.gumroad.com/l/okgucl
- `agent_token`: random hex, shared only with the production side
  (Muse's order-production step). Never commit the real token.
  (The local `web/fashion_config.json` already carries the real
  product ID, buy URL and a fresh token; reuse it or generate a new
  one on the box — just keep exactly one value in both places.)

## systemd unit — /etc/systemd/system/walk-web.service

```ini
[Unit]
Description=Fashion Citywalk web app
After=network.target

[Service]
WorkingDirectory=/home/ubuntu/walk-web
Environment=WALK_PORT=8796
ExecStart=/usr/bin/python3 /home/ubuntu/walk-web/app.py
Restart=always
RestartSec=3
User=ubuntu

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now walk-web
curl http://127.0.0.1:8796/walk/health   # {"ok": true}
```

## Caddy (api.newkwlab.com site block)

Add next to the existing `/duo*`, `/book*` and `/clip*` handles (keep
the BingSiteAuth.xml handle untouched):

```caddy
handle /walk* {
    reverse_proxy 127.0.0.1:8796
}
```

```bash
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
curl https://api.newkwlab.com/walk/health
curl https://api.newkwlab.com/walk/llms.txt
```

## Production flow (for reference)

1. `GET /walk/agent/paid` with header `X-Agent-Token: <agent_token>`
   → paid orders with template + template_label + name + photo URL.
2. `GET /walk/agent/orders/<id>/photo/1` (same header) → the buyer's
   photo (solo — always exactly one photo per order).
3. Produce the film per the paper-cutout fashion pipeline in the
   order's template (master cutout → front + profile A/C/B/C walk
   poses → template scene stickers → pop-groove music).
4. `POST /walk/agent/deliver/<id>` (same header), multipart form with
   `mp4` (the finished film) → order becomes `delivered`; buyers
   download from their order page link
   (`/walk/files/<id>/<delivery_token>/film.mp4`).

## Data hygiene

- Unpaid (`received`) orders auto-delete after 7 days; all order data
  after 60 days (same pattern as duo/book/clip).
- Buyer photos live only inside `data/orders/<id>/` — the studio's
  working copy is deleted after delivery per the site privacy copy.
