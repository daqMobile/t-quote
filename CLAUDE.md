# Quick Quote: project guide for Claude

Read this first in a new chat about Quick Quote. It replaces re-reading old conversations.

## What it is
- **Quick Quote**: a T-Mobile in-store quoting tool used by Nik, a T-Mobile retail rep in Saint Louis, on an iPad Pro 13" (landscape 1366×1024, portrait 1032×1376). Live at https://daqmobile.github.io/t-quote/ (GitHub Pages from `main`).
- Repo `daqMobile/t-quote`: `index.html` (the whole app: HTML, CSS, JS in one file) and `promos.json` (live offers, device prices, prepaid and business data). Saved quotes go to `daqMobile/t-quote-data`.
- Nik pushes nothing himself: commit and push straight to `main` (`git pull -q` first, then `git push -q origin HEAD:main`).

## Working with Nik
- Visual person: send screenshots (Playwright, deviceScaleFactor 2) and use tables.
- **Keep usage low.** Don't over-research; prefer facts he gives (screenshots from the store system) over web lookups. Batch small edits.
- Ask before big or ambiguous builds; otherwise just do it and show the result.
- Never send his email address to outside services. Quote emails go out from his personal Outlook through the Outlook app.
- Plain, short replies. No recaps of steps.

## App structure (index.html)
- State in localStorage: consumer `tmix-quick-quote.v2`, business `tmix-quick-quote.v2.biz`, device-price memory `…v2.prices`. The app always opens on the consumer screen (Nik's choice); the business quote waits until he switches. `normalizeState()`, `restoreState()`, `applyQuote()` (saved quotes open on their own screen).
- `compute(state)` → `{c, W (monthly rows), H (internet rows), T (due today), ONE (one-time credits), RB (rebate cards), monthly, …}`. `ctxOf(s)` gives `c.fam` = `tmix` | `exp` | `biz`, `c.pid`, line counts.
- Plans: `TMIX`, `EXP`, `LEGACY` (consumer retired: Go5G Next/Plus, Go5G, Essentials; upgrade quotes only via `isLegacy`), `BIZ` (SuperMobile/ProMobile/CoreMobile + older business plans with `legacy:true`), `CPLANS` / `CPLANS_BIZ` (tablet, watch), `planCalc()`, `legacyRef()`.
- Offers: `offerPool()` (consumer `live.offers || OFFERS`; business `live.bizOffers || BIZ_OFFERS`), `offersFor()`, BYOD (`byod`, `pay:"card"`), plan promotions (`pay:"plan"`, `planMo`, `planMonths`), AWAY (`product:"away"`). Plan discounts are account-wide only (Military, First Responder, 55+, Work Perks, Employee).
- Screens: Quick Quote (consumer) and Quick Quote for Business, switched from the top-left menu (`switchMode()`, `data-mode="biz"` = darker magenta). Tabs: Wireless, Home/Business Internet, Prepaid (consumer only; sub-tabs T-Mobile Prepaid with a prepaid quote, and Connect by T-Mobile).
- Side panels via `drawer()` / `renderSheet()`; slide animations for review, compare, hcompare, visa, email, save.
- Settings → Device prices: last confirmed date, "Check device prices now" link (`PRICE_CHECK_URL`), and price changes Quick Quote spotted (`trackPrices()`).

## Daily data pipeline
- **"Quick Quote promo research"** (scheduled task `trig_016HvyXxhoRMmKjMn8rTjGET`, 6:39 AM CT, editable with `update_trigger`; its prompt must be resent in full). Promotions daily (consumer, business, prepaid light); second read only for offers that changed; device prices Mondays only; edits promos.json with scripts; writes it to the hand-off page https://claude.ai/artifact/LZ4d2GrTtBEvpGVemVXQz1. Run message "DEVICE PRICE CHECK ONLY" = device prices only, then fires the publish task.
- **"Quick Quote promo check"** (`trig_019QSdYBAhA8mWhXGrVKJ3eX`, 7:30 AM, created via API so it can't be edited from chat): publishes the hand-off promos.json to the repo.
- **Device Price Check page** https://claude.ai/artifact/UoxzuYmr1eFpsaqAZZ934p: one button that fires the research task in device-only mode (mcp capability, Claude Code Remote).
- Plan prices live in index.html and are never checked by the pipeline (retired plans never); only promotions and device prices are.
- promos.json format: top-level keys meta, offers, devices, prepaid, business; keys sorted inside objects; offers/devices/business.offers sorted by id; indent 1; non-ASCII as-is; trailing newline.

## Testing
- Syntax: extract the largest `<script>` from index.html to a scratch `app.js`, then `node --check`.
- Behavior: Playwright Chromium against a tiny local `http` server on a free port serving the repo folder (serve a modified promos.json to test data changes). Check for `pageerror`s. Use iPad sizes above; phone 390×844.
- Known: at phone width the header and plan tier buttons scroll sideways (pre-existing).

## Open questions (ask Nik before changing)
- Business: 6+ line AutoPay handling is inferred from T-Mobile's ads; wearable line price unpublished ($10 placeholder, flagged on quotes); business device prices can differ from consumer list prices; Experience for Business / Ultimate+ need full price grids to quote new lines.
- Watch Plan Plus Canada/Mexico data (5GB vs 15GB) and the Home Internet voice-line discount ($20 vs $15) are unconfirmed.
- Quote email subject: offer to drop "T-Mobile" from the default subject (spam filtering) was never answered.
