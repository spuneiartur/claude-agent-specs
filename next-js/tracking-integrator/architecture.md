# Architecture map — consent + tracking per project

Read the section for the project you detected in Phase 0. The systems are functionally equivalent (3 categories: necessary / functional / analytics; Google Consent Mode v2; consent-gated tags) but live in different files and use different conventions. Never assume one project's layout in another.

## Differences at a glance

| Concern | **guan** | **megadoor** |
|---|---|---|
| Consent state | `hooks/use-cookie-consent.js` (Context defined in a hook file) | `contexts/CookieConsentContext.jsx` |
| Import path | `@hooks/use-cookie-consent` | `contexts` (barrel) |
| Cookie config | `constants/cookie-config.js` | `config/cookie-config.js` |
| localStorage key | `guan_cookie_consent` | `cookie_consent_preferences` |
| Consent → signals | `functions/consent-signals.js` (pure: returns a signals object) | `functions/update-gtm-consent.js` (calls `gtag('consent','update',…)`) |
| Banner folder | `components/CookieBanner/` | `components/CookieConsent/` |
| Consent Mode v2 default | `data/gtm.js` → `gtmConsentDefaultScript`, injected in `pages/_document.js` head | inline inside `components/GoogleTagManager.jsx`, injected in `pages/_document.js` head |
| GTM load | `next/script` afterInteractive in `pages/_app.js` | inline `dangerouslySetInnerHTML` in `_document` + `GoogleTagManagerNoscript.jsx` in `<body>` |
| Tag snippet location | extracted to `data/<vendor>.js` (string export) | inlined in the component |
| Browser event helper | `lib/meta-pixel.js` (default-export object `{ track }`, returns `eventID`) | none yet |
| Policy page | **data-driven**: `pages/{privacy,cookies,terms}.js` → `LegalSection` over arrays in `data/legal.js` | **hardcoded JSX**: `components/Visitor/TermsContent.jsx` at route `/termeni-si-conditii` |
| Meta Pixel | present (`components/MetaPixel.jsx` + `data/meta-pixel.js` + `lib/meta-pixel.js`) | not present |
| Has `cookie-consent-gdpr` skill | check `skills/` | yes |

When seeding a NEW project, prefer the **guan** shape — it's the cleaner/newer pattern (extracted snippets, pure signal mapper, data-driven policy pages).

## guan — file map

- **Consent provider/hook** — `hooks/use-cookie-consent.js`. Exports `CookieConsentProvider` + `useCookieConsent()`. Hook API: `{ consent, hasConsented, isLoading, acceptAll, rejectAll, saveConsent, resetConsent, isAllowed }`. `isAllowed('analytics')` is the gate you call. On save it persists to localStorage, dispatches `cookieConsentUpdated`, and calls `window.gtag('consent','update', consentSignals(consent))`.
- **Cookie config** — `constants/cookie-config.js`. Exports `CONSENT_STORAGE_KEY`, `DEFAULT_CONSENT` (`{necessary:true, functional:false, analytics:false}`), `CONSENT_V2_SIGNALS` (signal→category map), `COOKIE_CATEGORIES` (the operable list the banner renders; each category has `cookies:[{name,purpose,duration}]`). **Add new cookies here.**
- **Signal mapper** — `functions/consent-signals.js` (default `consentSignals`), exported via `@functions`. Pure: turns the consent object into `{ad_storage:'granted'|'denied', …}` per `CONSENT_V2_SIGNALS`. No change needed unless you add a category.
- **Consent Mode v2 default** — `data/gtm.js` → `gtmConsentDefaultScript`, rendered in `pages/_document.js` `<Head>` BEFORE `gtm.js`. Reads localStorage so returning visitors don't flash to denied.
- **Banner** — `components/CookieBanner/` (SimpleBanner, DetailedSettings, CookieCategoryCard renders the per-cookie details from `COOKIE_CATEGORIES`, CookieToggle, CookieSettingsButton, ConsentGate, index barrel).
- **Tag components (mounted in `pages/_app.js` inside `<CookieConsentProvider>`):**
  - `components/GoogleTagManager.jsx` + `data/gtm.js` (`gtmScript`). Loads unconditionally when `GTM_ID` set; gated by Consent Mode, not a client gate.
  - `components/MetaPixel.jsx` + `data/meta-pixel.js` (`metaPixelScript`) + `lib/meta-pixel.js` (`metaPixel.track`). Client-gated on `analytics`; re-fires `PageView` on `router.events` route changes; events carry `eventID`.
  - `components/MicrosoftClarity.jsx` + `data/clarity.js` (`clarityScript`). Client-gated on `analytics`. Env var `CLARITY_ID`.
- **Env** — `next.config.js` `env: {}` (A-Z). `.env`, `.env.example` (placeholder), `.env.template` (empty).
- **Policy pages (data-driven — THIS is where disclosures go):**
  - `pages/privacy.js`, `pages/cookies.js`, `pages/terms.js` each map `LegalSection` over a content array from `data/legal.js` (`privacyContent`, `cookieContent`, `termsContent`).
  - `components/LegalSection.jsx` renders `{title, text, list?, footer?}`.
  - `data/legal.js` is the editable content. Cookie policy references trackers; the authoritative per-cookie list is `constants/cookie-config.js`. Contact form links to `/privacy`. Footer links `/privacy`, `/terms`, `/cookies`.

## megadoor — file map

- **Consent provider/context** — `contexts/CookieConsentContext.jsx`, imported as `import { useCookieConsent } from 'contexts'`. API adds `updateConsent(category, value)`. localStorage key `cookie_consent_preferences`. On change calls `updateGtmConsent(consent)` and dispatches `cookieConsentUpdated`.
- **Cookie config** — `config/cookie-config.js`: `COOKIE_CATEGORIES`, `DEFAULT_CONSENT`, `CONSENT_STORAGE_KEY`. **Add new cookies here.**
- **Signal mapper** — `functions/update-gtm-consent.js` (`updateGtmConsent`): imperatively calls `window.gtag('consent','update',{ ad_storage: value(consent.analytics), … })`.
- **Consent Mode v2 default + GTM** — both in `components/GoogleTagManager.jsx` (default `consent` denied + the GTM loader), injected in `pages/_document.js` `<Head>`; `GoogleTagManagerNoscript.jsx` in `<body>`.
- **Banner** — `components/CookieConsent/` (CookieBanner, SimpleBanner, DetailedSettings, CookieCategoryCard, CookieToggle, ConsentGate, CookieSettingsButton). `ConsentGate category="functional"` wraps embeds (TikTok/social).
- **Tag components:**
  - `components/GoogleAnalytics.jsx` — GA4, client-gated on `analytics`, env `NEXT_PUBLIC_GA4_ID`. (Exists; verify whether it's mounted in `_app.js` before assuming GA4 is live.)
  - `components/MicrosoftClarity.jsx` — inline IIFE (no data file), env `CLARITY_ID`, client-gated, mounted in `_app.js`.
  - No Meta Pixel — adding one means porting guan's `MetaPixel.jsx` + `data/meta-pixel.js` + `lib/meta-pixel.js` pattern (inline the snippet to match megadoor style, or introduce a `data/` file — match whatever the project does at that time).
- **Env** — `next.config.js` `env: {}`.
- **Policy page (hardcoded — disclosures go in JSX):** `pages/termeni-si-conditii.js` → `components/Visitor/TermsContent.jsx`, hardcoded Romanian JSX. Currently documents Microsoft Clarity explicitly but does NOT enumerate every tracker — when you add a tag, add a matching section here.
- **Skill** — `skills/skills/cookie-consent-gdpr/` documents wiring tags into this system; read it for megadoor-specific guidance and keep it in sync (workspace CLAUDE.md rule).

## Other siblings (spunei, homespa, societyhostess, …)

Not yet mapped. Run Phase 0 discovery; they'll match one of the two shapes above (most newer ones follow guan). State which shape you found before editing.
