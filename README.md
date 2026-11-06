# Fashion Citywalk — Paper-Cutout Animation

Personalized paper-cutout citywalk animation from one photo.

- Live order site: https://api.newkwlab.com/walk/
- Gumroad product: Personalized Fashion Citywalk Paper-Cutout Animation ($12.99, Solo)
- Six templates: 01 Fashion Citywalk / 02 Tokyo Citywalk / 03 Shanghai Citywalk (Café) / 04 Off Duty Collage / 05 London Citywalk Duo / 06 Hong Kong Citywalk (Market)

## Layout

- `web/app.py` — order server (stdlib only, mirrors the SnapDuo/Storybook/PaperStage pattern: JSON orders, Gumroad license check fail-closed, agent token endpoint)
- `web/site/` — storefront + order desk (`index.html`, `llms.txt`, `media/` sample films and posters)
- `fashion_config.example.json` (repo root) — config template (placeholders only)
- `deploy/README.md` — EC2 deploy notes (walk-web.service, port 8796, Caddy `/walk*`)
- `SPEC.md` / `BUILD-REPORT.md` — build spec and report
- `REDESIGN.md` / `SWAP-REPORT.md` — poster-board redesign and template-swap notes

## Notes

- `web/fashion_config.json` (live config with the agent token) is intentionally NOT in this repo. Copy `fashion_config.example.json` and fill in real values on the server.
- Sample films (MP4) and posters live in `web/site/media/` on the server and are not stored in this repo (large media stays out of GitHub by rule).
- Production: EC2 `walk-web.service`, Caddy `/walk*` handle on `api.newkwlab.com`.
