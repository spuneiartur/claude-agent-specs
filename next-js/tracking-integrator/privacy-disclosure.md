# Privacy disclosure — updating the policy page

Artur's standing rule: **every new tracking integration must be reflected in the user-facing privacy/cookie policy**, using the vendor's _official_ disclosure documentation. This is two distinct updates — don't conflate them:

1. **Cookie config** (`cookie-config.js`) — the operable, per-cookie list the consent banner renders. Always update this.
2. **Policy page text** — the human-readable legal disclosure. Structure differs per project (below).

WebFetch the vendor's disclosure doc (see `vendor-playbooks.md` for URLs) and adapt _their_ required wording. Do not invent legal language; do not copy a competitor's. Keep the language consistent with the page's existing tone and locale (guan policy pages are English; megadoor is Romanian).

## Step 1 — Declare cookies (both projects)

In `COOKIE_CATEGORIES` (guan: `constants/cookie-config.js`; megadoor: `config/cookie-config.js`), add an entry to the correct category's `cookies` array:

```js
{ name: '_xxx', purpose: '<Vendor> <what it does>', duration: '<n months>' },
```

Use the real cookie names/durations from the vendor's cookie doc. This keeps the banner's "cookie details" accordion truthful — it's read directly from here.

## Step 2 — Update the policy page

### guan (data-driven — edit `data/legal.js`)

The policy routes `pages/privacy.js`, `pages/cookies.js`, `pages/terms.js` render `LegalSection` over arrays exported from `data/legal.js` (`privacyContent`, `cookieContent`, `termsContent`). To disclose a new vendor:

- Edit the relevant array (usually `cookieContent` for trackers, `privacyContent` for data-sharing/processors).
- A section object is `{ title, text, list?: [], footer?: '' }`. Either extend an existing "Third-Party Cookies" / "Data Sharing" section's `text`/`list`, or add a new numbered section for a significant processor (e.g. a dedicated "Microsoft Clarity" or "Meta Pixel" section, mirroring how megadoor calls out Clarity).
- Name the vendor, what it collects, the purpose, that it loads only after analytics consent, and link to the vendor's privacy statement.
- Do NOT hand-duplicate the full cookie table in prose — `cookie-config.js` (surfaced in the banner) is the authoritative per-cookie list. Keep the policy text at the "who/what/why + link" level.

### megadoor (hardcoded — edit `components/Visitor/TermsContent.jsx`)

Route `/termeni-si-conditii`. The disclosures are hardcoded Romanian JSX. Add a section mirroring the existing **MICROSOFT CLARITY** block: an `<h2>` with the vendor name + a `<p>` covering what it does, that it loads only after analytics consent (`...numai după ce vă exprimați acordul pentru cookie-urile de analiză...`), and a link to the vendor's privacy statement. Match surrounding markup and tone.

## Per-vendor disclosure essentials

Each disclosure should cover: **who** (vendor name + that it's a third party), **what** (data/cookies collected), **why** (purpose: analytics, heatmaps, advertising, measurement), **consent** (loads only after the analytics category is granted), and a **link** to the vendor's own privacy statement.

- **Microsoft Clarity** — _required_ by Microsoft: state you use Clarity & Microsoft to capture behavioral metrics/heatmaps/session replay; link <https://privacy.microsoft.com/privacystatement>. Doc: <https://learn.microsoft.com/en-us/clarity/setup-and-installation/privacy-disclosure>
- **Meta Pixel** — disclose use of Meta business tools / pixel and that event data is shared with Meta for advertising & measurement; link Meta's data policy. Doc: <https://www.facebook.com/legal/terms/businesstools>
- **Google (GTM/GA4/Ads)** — disclose analytics/advertising via Google and link Google's privacy & partner-sites pages. Docs: <https://business.safety.google/privacy/> , <https://policies.google.com/technologies/partner-sites>
- **TikTok / LinkedIn / Hotjar** — disclose the tool, its purpose, and link its privacy/cookie policy (URLs in `vendor-playbooks.md`).

## Checklist before you call a tracking integration "done"

- [ ] Cookie(s) added to `COOKIE_CATEGORIES` (right category, real names/durations)
- [ ] Policy page updated (guan: `data/legal.js`; megadoor: `TermsContent.jsx`) with vendor name, purpose, consent note, privacy-statement link — wording sourced from the vendor's disclosure doc
- [ ] Tag is consent-gated and does not fire before consent
- [ ] Env var documented in `.env.example` / `.env.template`
- [ ] User told to restart `next`, accept the relevant consent category, and verify in DevTools + the vendor's helper
