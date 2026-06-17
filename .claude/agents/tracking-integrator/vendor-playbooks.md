# Vendor playbooks

One block per vendor: consent category, env var, script source, gating method, cookies it sets, and the **official privacy-disclosure doc** to WebFetch before writing policy text. Snippet versions drift — when in doubt, WebFetch the vendor's install doc for the current snippet rather than pasting from memory.

> Disclosure URLs below are entry points. Always WebFetch the live page and quote/adapt the vendor's own required wording — never invent disclosure language. See `privacy-disclosure.md` for where the text goes.

---

## Microsoft Clarity (heatmaps + session recording)

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`)
- **Env:** `CLARITY_ID` · **Script:** IIFE loading `https://www.clarity.ms/tag/<ID>`
- **Cookies:** `_clck`, `_clsk` (and `CLID` on the clarity.ms domain). Purpose: heatmaps & session recordings. Duration: `_clck` ~1 year, `_clsk` ~1 day.
- **Disclosure doc (canonical example):** https://learn.microsoft.com/en-us/clarity/setup-and-installation/privacy-disclosure — Clarity *requires* you to disclose, in your privacy notice, that you use Clarity/Microsoft to capture behavioral metrics, heatmaps, and session replay, and to link to Microsoft's privacy statement (https://privacy.microsoft.com/privacystatement).
- **Notes:** Reference shape for every other client-gated tag. guan extracts the snippet to `data/clarity.js`; megadoor inlines it.

## Google Tag Manager (container)

- **Category:** analytics (via Consent Mode, not a client gate) · **Gate:** Consent Mode v2 defaults (denied) until `gtag('consent','update',…)`
- **Env:** `GTM_ID` (`GTM-XXXX`) · **Script:** standard GTM loader + `<noscript>` iframe (megadoor includes the noscript; guan currently omits it — add it only if asked).
- **Cookies:** GTM itself sets none; tags *inside* the container do (GA4: `_ga`, `_ga_*`; Google Ads: `_gcl_*`, `IDE`/`test_cookie` on doubleclick). Disclose based on what the container actually fires — ask which tags are configured if unknown.
- **Disclosure doc:** https://business.safety.google/privacy/ + how Google uses partner-site data https://policies.google.com/technologies/partner-sites
- **Notes:** The Consent Mode v2 *default* script MUST run before the GTM loader (guan: `data/gtm.js` `gtmConsentDefaultScript` in `_document`). A Meta/TikTok pixel may already live inside the GTM container — check before adding a hardcoded one, or you'll double-fire (this exact situation occurred on guan: a `1486…` Meta pixel fired from GTM while the codebase had none).

## Google Analytics 4 (direct, not via GTM)

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`) OR Consent Mode if loaded through GTM
- **Env:** `GA4_ID` / `NEXT_PUBLIC_GA4_ID` (`G-XXXX`) — match project convention · **Script:** `https://www.googletagmanager.com/gtag/js?id=<ID>` + inline `gtag('config', …)`
- **Cookies:** `_ga`, `_ga_<container>`, `_gid`. Analytics/measurement.
- **Disclosure doc:** https://support.google.com/analytics/answer/7318509 (data practices) + Google privacy links above.
- **Notes:** If GTM already loads GA4, do NOT add a second direct GA4 tag.

## Meta (Facebook) Pixel

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`)
- **Env:** `META_PIXEL_ID` (numeric) · **Script:** `fbevents.js` IIFE → `fbq('init', '<ID>'); fbq('track','PageView');`
- **Cookies:** `_fbp` (~3 months, advertising), `_fbc` (~3 months, ad click attribution — set when an `fbclid` is present).
- **Disclosure doc:** https://developers.facebook.com/docs/meta-pixel + Meta Business Tools Terms https://www.facebook.com/legal/terms/businesstools and Cookie use https://www.facebook.com/policies/cookies/ — you must disclose use of Meta business tools / pixel and that data is shared with Meta for advertising & measurement.
- **Notes (guan pattern — replicate):**
  - Browser events go through `lib/meta-pixel.js` (`metaPixel.track(event, params)`), which **generates a unique `eventID`, passes it as `{ eventID }`, and returns it**. Always keep this even before the Conversions API exists — it's what makes server-side dedup a drop-in later (same `event_id` browser + server → Meta deduplicates).
  - Fire conversions from the API/mutation **success** handler, not the click: `CompleteRegistration` on signup, `Lead` on contact/lead forms, `Purchase`/`InitiateCheckout` on checkout.
  - Re-fire `PageView` on `router.events` `routeChangeComplete` (SPA nav) — the base snippet only fires the first one.
  - Conversions API (server-side) is a planned follow-up: same events, sent from the backend with the matching `event_id`. The pixel's `access token` is a server-only secret — never expose it via `next.config.js` env.

## TikTok Pixel

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`)
- **Env:** `TIKTOK_PIXEL_ID` · **Script:** TikTok `ttq` loader → `ttq.load('<ID>'); ttq.page();`
- **Cookies:** `_ttp` (~13 months, advertising).
- **Disclosure doc:** TikTok Business Products (Data) Terms https://www.tiktok.com/legal/page/global/bc-data-terms/en + privacy https://www.tiktok.com/legal/page/global/privacy-policy/en
- **Notes:** `cookie-config.js` may already list `_ttp` (guan does) — verify before duplicating. Distinguish the **pixel** (tracking, analytics category) from **embeds** (functional category, `ConsentGate`).

## LinkedIn Insight Tag

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`)
- **Env:** `LINKEDIN_PARTNER_ID` · **Script:** `_linkedin_partner_id` + `https://snap.licdn.com/li.lms-analytics/insight.min.js`
- **Cookies:** `li_sugr`, `bcookie`, `lidc`, `UserMatchHistory` (advertising/analytics).
- **Disclosure doc:** https://www.linkedin.com/legal/l/cookie-table + https://www.linkedin.com/legal/privacy-policy

## Hotjar (heatmaps/recordings — alternative to Clarity)

- **Category:** analytics · **Gate:** client (`isAllowed('analytics')`)
- **Env:** `HOTJAR_ID` (+ optional `HOTJAR_SV`) · **Script:** `hj` loader → `static.hotjar.com/c/hotjar-<ID>.js`
- **Cookies:** `_hjSessionUser_*`, `_hjSession_*`, etc.
- **Disclosure doc:** https://help.hotjar.com/hc/en-us/articles/115011789248-Hotjar-Cookies + https://www.hotjar.com/legal/policies/privacy/

## Google Ads (conversion / remarketing, standalone)

- **Category:** analytics · **Gate:** Consent Mode (preferred, via GTM) or client gate
- **Env:** `GOOGLE_ADS_ID` (`AW-XXXX`) · **Script:** gtag with the `AW-` id; conversions via `gtag('event','conversion',{send_to:…})`
- **Cookies:** `_gcl_au`, `IDE`, `test_cookie` (doubleclick.net).
- **Disclosure doc:** Google privacy/partner-sites links (see GTM block).

---

## Adding a vendor not listed here

1. WebSearch the official "install the <vendor> pixel/tag" doc for the current snippet + the cookies it sets.
2. WebFetch the vendor's **privacy/cookie disclosure** doc for required wording.
3. Default to: analytics category, client gate, raw snippet in `next/script afterInteractive`, no npm package.
4. Follow the 8-step workflow in the main agent file; declare cookies in `cookie-config.js`; update the policy page.
