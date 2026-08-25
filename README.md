# Boca da Encumeada — Website

Marketing website for Snack Bar Restaurante Boca Da Encumeada, a family-run mountain-pass restaurant in Serra de Água, Madeira.

**Live site:** https://bocadaencumeada.com (and https://www.bocadaencumeada.com)

## What's here

- `index.html` — the whole site. Single-file static HTML/CSS/vanilla JS, no build step, no framework. Bilingual (EN/PT) via a `content` dictionary + `data-i18n` attributes + `setLang()`.
- `images/` — the four processed photos (hero, terrace, railing, storefront). See `HANDOFF.md` for the exact ImageMagick crop recipe if these ever need regenerating from the originals.
- `wrangler.jsonc` — Cloudflare Workers (static assets) deploy config.
- `dist/` — deployable copy of `index.html` + `images/` (this is what `wrangler deploy` actually uploads, kept separate so `wrangler.jsonc` itself doesn't get served as a static asset). **After editing `index.html` or `images/`, copy them into `dist/` before deploying.**
- `HANDOFF.md` — the full project handoff/master-plan doc: research, design decisions, deploy method, gotchas, and a progress checklist. Read this first if picking this project back up.

## Deploy

```bash
cp index.html dist/index.html   # if index.html changed
export CLOUDFLARE_API_TOKEN="..."   # get from Cloudflare dashboard → Profile → API Tokens
export CLOUDFLARE_ACCOUNT_ID="6cc585c7a88587893be0a6d58fa83eca"
wrangler deploy
```

See `HANDOFF.md` for the full gotchas (token IP restrictions, custom domain binding via direct API call rather than wrangler routes, etc).

## Design reference

There's also a Claude Design canvas mockup (separate from this production build, used during early design iteration): https://claude.ai/code/artifact/cf0e01ca-861c-4c96-9f00-df95af6a4bd9
