# Boca da Encumeada Website — Master Plan / Handoff Doc

**Status as of 2026-09-12: full printed food + drinks menus with prices added to the site, audited, deployed — see the 2026-09-12 section. Previous: 2026-08-28 (late): 🟢 LIVE at bocadaencumeada.com with the full site rehaul deployed (see "2026-08-28 rehaul" section below). Hours match the real Google Business listing (Mon–Sat 8:30–19:30, closed Sundays). Mac folder + GitHub repo `github.com/leonfigueira/boca-da-encumeada-site` in sync with the live site. The full, authoritative version of this doc is the Claude Project doc `claude/boca-da-encumeada-website-masterplan.md`.**

This doc is mirrored as a Claude Project doc (`claude/boca-da-encumeada-website-masterplan.md`), and also as `~/Desktop/Projects/App-Handoffs/10-BOCA-DA-ENCUMEADA.md`.

## Where things stand right now

- ✅ **LIVE PRODUCTION SITE:** https://bocadaencumeada.com (and `www.` variant) — Cloudflare Worker with static assets, Custom Domain bound on both apex and www.
- ✅ Bilingual EN/PT, real high-res photos, real TripAdvisor review quotes, "About Us" blurb, mobile-specific hero/gallery/panorama fixes, correct hours.
- ✅ Local backup on Leon's Mac: `~/Desktop/Projects/Boca-da-Encumeada-Site/`.
- ✅ **Pushed to GitHub:** https://github.com/leonfigueira/boca-da-encumeada-site — branch `main`, up to date with the live site.

## 2026-08-28 update: hours fix + new photos

Leon sent a screenshot of the actual Google Business listing, which showed the site had the wrong hours the whole time: it said "closed Mondays, 8:00–20:00" when the real listing is **closed Sundays, Monday–Saturday 8:30–19:30**. Fixed in every spot it appears in `index.html`: meta description, hours strip, Visit section, footer, EN+PT content dictionaries (7 occurrences — search `hoursStrip`/`hoursTitle`/`hoursCaption`/`footerClosed`). Dropped the "kitchen closes ~19:00" caption since it wasn't part of the verified source.

Added two new photos — espresso from Cafés TOFA and a pair of house specialty drinks, both portrait/"skinny" shots. Placed in a new `.duo-photos` flex row after the existing gallery, sized by fixed height with `width: auto` so each keeps its natural aspect ratio rather than being force-cropped. Saved as `images/coffee.jpg` and `images/drinks.jpg` (960px tall, quality 85). Verified with local Playwright screenshots before deploying.

## GitHub access — root cause and the actual fix (resolved 2026-08-25)

**What didn't work:** pushing directly from a cloud Cowork session (`git`/`gh` inside the container) is blocked by an Anthropic-side proxy that injects its own credentials regardless of token supplied — a PAT does not fix this. The `device_bash` device-bridge sandbox has no network route to github.com at all. The claude.ai chat "Add from GitHub" connector links repos fine but 404s on content access (confirmed Anthropic bug, GitHub issue `anthropics/claude-code#71542`). Browser automation through GitHub's web UI works but is slow and token-expensive.

**The actual fix:** `mcp__remote-devices__Desktop_Commander__start_process` gives a real shell on Leon's actual Mac, and it already has `gh` authenticated as him via the OS keyring — no proxy involved. So:
```bash
cd ~/Desktop/Projects/Boca-da-Encumeada-Site
git remote add origin https://github.com/leonfigueira/boca-da-encumeada-site.git   # only if not already set
git branch -M main
git push -u origin main
```
This is the standing answer for any future session: don't fight the cloud session's GitHub access — run `git`/`gh` on Leon's Mac via `Desktop_Commander__start_process` instead.

## Deploy method (Cloudflare Workers + static assets)

1. **Cloudflare API Token — use this exact recipe:**
   Go to **dash.cloudflare.com/{account_id}/api-tokens** (the **"Account API tokens"** page under Manage Account — NOT "My Profile → API Tokens") → **Create Token** → click the **"Edit Cloudflare Workers"** quick-permission-policy button → name it, leave **Client IP address filtering** on its default **"Allow"** → **Review token → Create token**. Under a minute, no dropdowns to fight with.
   - **Gotcha (2026-08-28), avoid this path:** "My Profile → API Tokens"'s "Edit Cloudflare Workers" template requires picking an Account Resources value from a react-select dropdown that's extremely resistant to browser automation. The Account API tokens page above has no such dropdown and defaults IP filtering to "Allow." Also: "Edit Cloudflare Workers" and "Workers AI" are separate, adjacent-looking templates — Workers AI is useless for deploying this site (confirmed: `wrangler deploy` fails with error 10000 against it).
2. Wrangler CLI (`npm install -g wrangler`, v4.125.0+). `wrangler.jsonc`:
   ```json
   {
     "name": "boca-da-encumeada",
     "compatibility_date": "2026-08-25",
     "workers_dev": false,
     "assets": { "directory": "./dist", "not_found_handling": "single-page-application" }
   }
   ```
3. `index.html` + `images/*.jpg` live in `dist/` (copied there before each deploy).
4. `wrangler deploy` uploads the Worker + assets. Custom domain binding (already done, only needed if removed) via direct API call:
   ```bash
   curl -X PUT "https://api.cloudflare.com/client/v4/accounts/{account_id}/workers/domains" \
     -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" -H "Content-Type: application/json" \
     -d '{"hostname":"bocadaencumeada.com","service":"boca-da-encumeada","environment":"production","zone_id":"a68505257839cc343588ca19d85f322a"}'
   ```
   Account ID: `6cc585c7a88587893be0a6d58fa83eca`.
5. Cloudflare token is never saved to disk — Leon generates a new one each session via the recipe above.
6. Local Playwright (headless Chromium) can screenshot `file://` paths fine for pre-deploy checks; it cannot reach the live HTTPS site from this sandbox (`ERR_CONNECTION_RESET`) — use `curl` to verify after deploying instead.

## Progress checklist

- [x] Research restaurant facts, menu, reviews, location story
- [x] Domain `bocadaencumeada.com` bought on Cloudflare
- [x] Design mockup → production static site
- [x] Deploy live to `bocadaencumeada.com` + `www.`
- [x] Mobile-audit round: hero zoom-out + text trim, gallery stacking, panorama full-bleed, About Us blurb
- [x] Local project folder on Leon's Mac, handoff doc mirrored
- [x] Pushed to GitHub via `Desktop_Commander__start_process` running real `git`/`gh` on Leon's Mac
- [x] Hours corrected to match the real Google Business listing
- [x] Coffee + drinks photo pair added, portrait-preserving layout
- [ ] Get Leon's eyes-on confirmation on the mobile-audit fixes specifically
- [ ] Optional: more languages, more reviews, live Google Reviews widget

---

## Reference detail

### Images — canonical processing recipe

- **hero.jpg**: from `IMG_8834.jpeg` (10346×3678). Crop right ~6%, resize 2400px wide, quality 87.
- **terrace.jpg**: from `IMG_8757.jpeg`. Auto-orient, crop top ~64%, resize 1500px, quality 84.
- **railing.jpg**: from `IMG_8837.jpeg`. Crop drops building + pole, resize 1500px, quality 84.
- **storefront.jpg**: from a phone photo. Crop drops car + clutter, resize 1500px, quality 84.
- **coffee.jpg**: espresso, portrait source 796×1728 → resized 960px tall, quality 85, natural aspect (no crop).
- **drinks.jpg**: two specialty drinks, portrait source 906×1966 → same recipe. Added 2026-08-28.

### Research findings (restaurant facts)

- **Name:** Snack Bar Restaurante Boca Da Encumeada
- **Address:** Estrada Regional da Encumeada, Terraço dos Temperos, Serra de Água, Ribeira Brava, Madeira, 9350-330, Portugal
- **Phone:** +351 291 952 319
- **Hours (corrected 2026-08-28, per Leon's Google Business listing):** Monday–Saturday 8:30–19:30, closed Sundays.
- **Rating:** 4.5/5 on TripAdvisor (41 reviews)
- **Real review quotes used** (kept in original English): Borek H. (Czech Republic) 5★, Jo (UK) 4★, James P. (UK) 4★, heftlee9901 (Miami) 5★, Mark R. 5★.

### Design direction

Warm Madeira mountain-lodge aesthetic: terracotta/stone tones, deep Laurisilva green (`--pine`), Cormorant Garamond (display) + Karla (body). Flow: sticky nav → Hero → hours/rating strip → About Us → Story → Menu → gallery → coffee/drinks duo → Reviews → Visit → panorama → Footer.

### Bilingual EN/PT implementation

Live toggle, `data-i18n` attributes + JS `content` dictionary. Real review quotes left in original English. European Portuguese, not Brazilian.

## 2026-08-28 rehaul (deployed, preview-first)

Leon asked for a contrarian review of the whole page (hero protected). What shipped:

1. **Bottom panorama was the identical file as the hero.** Replaced with `images/valley.jpg` — new crop of the same source panorama (`IMG_8834.jpeg` on Leon's Desktop): `-crop 5173x1960+350+600`, the left-side valley-to-the-sea view; the +350 offset excludes a power line. Same finish (2400px, Lanczos, unsharp, q87).
2. **Photo restructure:** drinks/coffee duo moved INTO the Menu section as figures with captions; railing shot became a full-bleed `.photo-band` between Menu and Reviews (`object-position: center 65%` — avoids the power pole; verified by screenshot comparison); storefront moved to Visit as a wayfinding photo.
3. **Visit is a two-column `.visit-grid`:** info + new drive-times row on the left; Google Maps embed (`.map-frame` iframe, no API key) + storefront on the right. ⏳ The map couldn't be visually verified from the sandbox — confirm it renders live.
4. **Honesty fixes:** reviews retitled ("What travellers say" — all quotes are TripAdvisor, none Google); both aggregate star displays render an actual 4½ (`.stars`/`.stars-bg`/`.stars-fill` overlap, width 90%).
5. **Menu fixes:** non-apologetic intro, distinct card icons (flame/tumbler/pot/bowl/cup), poncha card trimmed.
6. **Story P1 rewritten** to stop repeating the About blurb — now laurisilva + levada trailheads.
7. **Footer** shows full hours.
8. **The two milky drinks are NOT poncha (Leon confirmed)** — caption is "Something cold for the terrace" / "Algo fresco para a esplanada". Don't re-caption them as poncha.

New i18n keys (both dictionaries): `duoDrinksCaption`, `duoCoffeeCaption`, `driveTitle`, `driveCaption`, `visitPhotoCaption`. Every new user-facing string needs a key in BOTH the EN and PT dictionaries or it silently stays English when toggled.

**Current page flow:** nav → Hero → strip (4½ stars) → About → Story → Menu (+duo) → railing band → Reviews → Visit (info | map+storefront) → valley panorama → Footer.

## 2026-09-12: the printed table menu goes on the site (audited)

Leon's mum sent the printer's proof (`Maqueta_Vinil_Mesa.pdf`, ViProduções, 42.2×35.2 cm table vinyl, PT names + EN/FR/DE lines) and then photos of the printed vinyl. **The printed vinyl is the source of truth for dishes and prices.** It matches the proof exactly plus one addition after proofing: `Bife de Atum — 18,50 €` (first item under PEIXES). 71 items in 15 categories. No drinks on it at all.

**Customers are mostly tourists** (Leon, 2026-09-12) — English descriptions explain the Madeiran dishes; six local dishes carry a "Madeiran"/"Madeirense" tag (bolo do caco, espetada, both beef picados, espada, milho frito, bolo de mel).

What changed in `index.html` (all inside `#ementa` unless noted):
- **Data:** `var MENU = [...]` at the top of the `<script>` — `[PT name, EN line, price, madeiranTag?]` per item, category `{pt, en, note?}`. **Edit prices here only.** `renderMenu(lang)` (called from `setLang`) draws it into `#menu-list`; PT shows the PT name only, EN adds the English line. Category headers localised. When the vinyl is reprinted, update `MENU` and the "September 2026" date in `menuFoot` (EN + PT) the same day.
- **Cards:** six → three (Espetada & Picado / Bolo do Caco & Fondue / Espada & Bacalhau). Dropped: "Fresh Poncha" (no drinks on the menu), "Sandwiches & Soups" (claimed tomato and fish soup — not on the menu), "Galão & Homemade Cake" (coffee moved to the bar note), "Ask about today's specials — menu changes with the season and the catch" (unverifiable against a fixed printed menu).
- **Full list** in a `<details id="menu-details">` — open on desktop (`matchMedia(min-width: 861px)` in `init`), collapsed on phones (the list is ~7,000 px tall at 390 px). Footnote `menuFoot`: prices in euros / dated / kids + vegetarian / allergies → staff / phone (non-breaking spaces in the number).
- Photos moved after the list; `barNote` after the photos (galão, chinesa, passion-fruit juice, tap beer, poncha — all from the quoted reviews; no prices).
- **Poncha demoted, not removed:** About now leads with espetada and the view ("and a proper poncha for whoever isn't driving"); meta description no longer leads with poncha. Restore if the parents confirm poncha really is the signature (question sent 2026-09-12). Also asked: espetada on wood or gas — the site now avoids "over the open fire".
- Eyebrow "On the Grill" → "The Menu" / "A Ementa"; "Reviewers have called it the best on the island" → "one reviewer called our espetada the best they'd ever had" (that is what `review1` says).
- Pre-existing bug fixed in passing: the nav phone button wrapped onto four lines at 390 px; on phones it now shows the icon + "Call"/"Ligar" (`.nav-phone-num` / `.nav-phone-short`, key `navCall`).
- New i18n keys (both dictionaries): `menuListTitle`, `menuListHint`, `menuFoot`, `menuTag`, `barNote`, `navCall`. Removed: `card4*`, `card5*`, `card6*`.

**Process:** a contrarian auditor agent reviewed the plan (15 findings) and the built result (10 findings, zero data mismatches after parsing `MENU` against the transcription); everything actionable was applied. Verified with local Playwright screenshots at 1280 and 390 px, EN and PT, no console errors.

**The printed vinyl has translation errors** (not fixed from the proof; the proof's small print makes the client responsible for spelling): picado = "Minced Beef / Boeuf Haché / Gehacktes Rindfleisch"; CARNES header German "Omelett" (Fleisch); DOSES "Dosis" (Beilagen); "Foudue de Queijo / Cheese Foudue"; "Kabeljau Nach Hausarzt" (nach Art des Hauses); "Vorpeisen"; "Toasbrot"; "ceufs"; "Green Pepper / Steak de Filet Vert" (green peppercorn / au poivre vert); escalope sandwich called a salad in FR/DE; Fondue de Queijo e Alheira = "Cheese Fondue and Chicken"; "Fried Corn" for milho frito; "Sahnetorte" for pastel de nata; "Salaten"; "Salade Mixte – Petit/Grand". The site does NOT copy these, so the site and the table menu disagree on those lines until a reprint. Leon has a message to send his parents about it.

**Drinks (same day, from photos of the printed "Bebidas" card):** `var DRINKS = [...]` next to `MENU`, same shape (a string price like `'5,00 / 6,00 €'` is shown as-is — used for nikitas). Rendered by `renderList('drinks-list', DRINKS, lang)` into a second `<details id="drinks-details">` under the food list, open on desktop, collapsed on phones. 108 items: ponchas (9), nikitas (7), cocktails, fresh juices, coffee, teas, beer, wine/sangria, Madeira wine, port, liqueurs, soft drinks, water. Keys `drinksListTitle`, `drinksListHint`, `drinksFoot`; `barNote` removed (redundant now). Poncha IS on the drinks card (nine of them), so the About wording stands.

**Video (same day):** Leon sent a 12 s portrait reel (sunset over a sea of clouds, poncha raised). Encoded with ffmpeg, audio stripped (browsers only autoplay muted; the reel's audio track is a rights question anyway): `images/poncha-sunset.mp4` (H.264, ~550 KB) + `images/poncha-sunset.webm` (VP9 fallback) + `images/poncha-sunset-poster.jpg` (frame at 9.5 s). Placed as the FIRST figure in the `.duo-photos` row in the Menu section — `<video autoplay muted loop playsinline preload="metadata">`, 400 px tall / 220 px on phones, caption key `duoVideoCaption`. Note: headless Playwright screenshots show the tile black unless Chromium is launched with `--autoplay-policy=no-user-gesture-required`; that is the test browser, not the site.

**Not done, deliberately:** FR/DE site languages (the vinyl has the strings; the rest of the site would need translating — obvious next step for a tourist audience); a menu PDF download (the proof carries the printer's disclaimer).

