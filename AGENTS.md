# AGENTS.md — Boca-da-Encumeada-Site (bocadaencumeada.com)

Read `../AGENTS-CORE.md` first (absolute: `/Users/leon/Desktop/Projects/AGENTS-CORE.md`). That is
the working method. This file adds the specifics.

## What this is

- GitHub repo: `github.com/leonfigueira/boca-da-encumeada-site` (remote `origin`, branch `main`).
- Live URL: `https://bocadaencumeada.com` (and `https://www.bocadaencumeada.com`).
- The marketing site for Snack Bar Restaurante Boca da Encumeada, Leon's parents' family-run
  mountain-pass restaurant in Serra de Agua, Madeira. Audience is mostly tourists. Bilingual
  EN/PT.
- Stack: single-file static site. `index.html` is the whole site (HTML + CSS + vanilla JS), no
  framework, no build step. Bilingual via a `content` dictionary, `data-i18n` attributes, and
  `setLang()`. Served as Cloudflare Workers static assets. Full project handoff: `HANDOFF.md`.

## Prices are truth-sensitive (a real-world constraint)

- Menu prices live in the `var MENU = [...]` array at the top of the `<script>` in `index.html`,
  as `[PT name, EN line, price, madeiranTag?]`. The PRINTED table vinyl is the source of truth,
  not the code and not any earlier draft. Update `MENU` and the date in `menuFoot` (both EN and
  PT) only from the current printed menu, and only when Leon confirms.
- The printed vinyl has known translation errors. Do not silently "correct" or invent Portuguese
  or English wording; flag anything that looks wrong to Leon rather than changing it yourself.

## Deploy structure (verified on disk, from README/wrangler.jsonc)

- `dist/` is what `wrangler deploy` uploads: `dist/index.html` + `dist/images/`. It is kept
  separate so `wrangler.jsonc` is not served. AFTER editing `index.html` or `images/`, copy them
  into `dist/` before any deploy (`cp index.html dist/index.html`).
- Deploy (do NOT run this yourself, see below): `cp index.html dist/index.html` then
  `CLOUDFLARE_API_TOKEN=<token> CLOUDFLARE_ACCOUNT_ID=6cc585c7a88587893be0a6d58fa83eca wrangler deploy`.
  The custom domain is bound via a direct API call, not wrangler routes; token IP restrictions and
  the full gotchas are in `HANDOFF.md`.
- `images/` regeneration recipe (ImageMagick crops) is in `HANDOFF.md`.

## Web verification before any publish

- Open `index.html` (or `dist/index.html`) in a browser. Console clean, images load, `setLang()`
  toggles EN/PT correctly on every visible string, prices match the printed vinyl. Check mobile
  width (tourist audience is largely on phones).

## Publishing rule

`wrangler deploy` publishes to the live public site (Leon's parents' business). Do NOT deploy.
Prepare and verify, copy into `dist/`, show Leon the diff and any menu/price/copy change, and let
Leon deploy. Every publish and every customer-facing string is drafted and shown to Leon first.
No em dashes.
