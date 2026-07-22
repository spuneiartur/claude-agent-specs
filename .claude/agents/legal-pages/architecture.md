# Architecture map — legal pages + consent system per project

Read the section for the project you detected in Phase 0. All systems are functionally equivalent (necessary + analytics — sometimes + functional — categories; Google Consent Mode v2; consent-gated tags) but live in different files and use different conventions. Never assume one project's layout in another.

## Differences at a glance

| Concern | **guan** | **megadoor** | **homespa** | **createvelab** |
|---|---|---|---|---|
| Consent state | `hooks/use-cookie-consent.js` (Context in a hook file) | `contexts/CookieConsentContext.jsx` | `contexts/CookieConsentContext.jsx` | `hooks/use-cookie-consent.js` |
| Import path | `@hooks/use-cookie-consent` | `contexts` (barrel) | `@contexts` (barrel) | `@hooks/use-cookie-consent` |
| Cookie config | `constants/cookie-config.js` | `config/cookie-config.js` | `config/cookie-config.js` | `constants/cookie-config.js` |
| localStorage key | `guan_cookie_consent` | `cookie_consent_preferences` | `cookie_consent_preferences` | `createvelab_cookie_consent` |
| Consent → signals | `functions/consent-signals.js` (pure, returns signals object) | `functions/update-gtm-consent.js` (imperative `gtag` call) | `functions/update-gtm-consent.js` | `functions/consent-signals.ts` (pure) |
| Banner folder | `components/CookieBanner/` | `components/CookieConsent/` | `components/CookieConsent/` | `components/CookieBanner/` |
| Categories | necessary / functional / analytics | necessary / functional / analytics | necessary / functional / analytics | necessary / analytics (no embeds yet — add `functional` only if one is introduced) |
| Policy pages | **data-driven**: `pages/{privacy,cookies,terms}.js` → `LegalSection` over `data/legal.js` | **hardcoded JSX**: `pages/termeni-si-conditii.js` → `components/Visitor/TermsContent.jsx` (terms only — no separate privacy/cookies page) | **data-driven**: `pages/{privacy,cookies,terms}.js` → `LegalPage` wrapper + `components/Legal/{Privacy,Cookies,Terms}Content.jsx` (JSX per page, not array-driven) | **data-driven**: `pages/{termeni-si-conditii,politica-de-confidentialitate,politica-cookie-uri}.js` → `LegalSection` over `data/legal.js` |
| Routes | `/privacy` `/cookies` `/terms` (English site) | `/termeni-si-conditii` (Romanian site) | `/privacy` `/cookies` `/terms` (English site, RO legal refs) | `/termeni-si-conditii` `/politica-de-confidentialitate` `/politica-cookie-uri` (Romanian site) |
| Legal entity | TIM GLOBAL S.R.L. (SRL, has CIF + trade register) | — | (check `data/contact.js` before assuming) | Nichiforeac Evelina PFA, București (no CIF) |
| Tag components present | GTM, Meta Pixel, Microsoft Clarity | Google Analytics, Microsoft Clarity | check before assuming | none yet — cookie policy states this honestly |

When seeding a NEW project, prefer the **guan/createvelab shape** — hook-embedded context, pure signal mapper, data-driven policy pages, folders (`constants/`, `hooks/`, `functions/`) that already exist in most of Artur's Next.js starters.

## guan — file map

- **Consent provider/hook** — `hooks/use-cookie-consent.js`. Exports `CookieConsentProvider` + `useCookieConsent()`. API: `{ consent, hasConsented, isLoading, acceptAll, rejectAll, saveConsent, resetConsent, isAllowed }`.
- **Cookie config** — `constants/cookie-config.js`: `CONSENT_STORAGE_KEY`, `DEFAULT_CONSENT`, `CONSENT_V2_SIGNALS`, `COOKIE_CATEGORIES` (each category has `cookies: [{name, purpose, duration}]` — the operable source the banner's accordion reads).
- **Signal mapper** — `functions/consent-signals.js`, exported via `@functions`.
- **Banner** — `components/CookieBanner/` (SimpleBanner, DetailedSettings, CookieCategoryCard, CookieToggle, CookieSettingsButton, ConsentGate, index barrel).
- **Policy pages** — `pages/privacy.js`, `pages/cookies.js`, `pages/terms.js`, each mapping `LegalSection` over `privacyContent`/`cookieContent`/`termsContent` from `data/legal.js`. `LegalSection` renders `{title, text, list?, footer?}`.
- **Legal entity** — `data/legal.js` top: `companyInfo = { name, address, cif }` (TIM GLOBAL S.R.L.).

## megadoor — file map

- **Consent provider/context** — `contexts/CookieConsentContext.jsx`, imported `import { useCookieConsent } from 'contexts'`. Adds `updateConsent(category, value)`.
- **Cookie config** — `config/cookie-config.js`.
- **Signal mapper** — `functions/update-gtm-consent.js` (imperative `window.gtag('consent','update',...)`).
- **Banner** — `components/CookieConsent/` (same 7-file set as guan's `CookieBanner/`, different folder name). `ConsentGate category="functional"` wraps TikTok/social embeds.
- **Policy page** — only `pages/termeni-si-conditii.js` exists → `components/Visitor/TermsContent.jsx`, hardcoded Romanian JSX. No separate privacy/cookie routes as of last check — if asked to add them, follow the data-driven pattern from guan/createvelab rather than hand-writing more hardcoded JSX (less duplication, easier to keep the cookie inventory honest).

## homespa — file map

- **Consent provider/context** — `contexts/CookieConsentContext.jsx` (megadoor shape).
- **Cookie config** — `config/cookie-config.js`.
- **Banner** — `components/CookieConsent/`.
- **Policy pages** — `pages/{privacy,cookies,terms}.js`, each a thin page importing a shared `components/Legal/LegalPage.jsx` wrapper (title + hero + `Footer`) and a per-page content component: `components/Legal/PrivacyContent.jsx`, `CookiesContent.jsx`, `TermsContent.jsx`. Content is hand-written JSX with an inline `Section` subcomponent (`{title, children}`), not array-driven — slightly more duplication than guan's pattern but easier to write rich inline links/formatting.
- **Custom CSS** — `css/legal.css` styles `.legal-content` (bullet markers, `<strong>` emphasis) — the one legitimate custom-CSS use case here since bespoke bullet styling isn't expressible in Tailwind utilities alone. Only replicate this in a new project if its Tailwind config can't express the same bullet via `list-*` utilities.

## createvelab — file map (seeded by this agent, 2026)

- **Consent provider/hook** — `hooks/use-cookie-consent.js` (guan shape, no `functional` category).
- **Cookie config** — `constants/cookie-config.js`. `COOKIE_CATEGORIES` currently has empty `cookies: []` for `analytics` — no tracker wired yet; **when `tracking-integrator` adds one, that array must be filled in** (this is the seam between the two agents).
- **Signal mapper** — `functions/consent-signals.ts` (project's `functions/` folder is TypeScript; other projects' are `.js`).
- **Banner** — `components/CookieBanner/`, restyled with the project's actual palette (`primary`/`secondary`/`accent` from `tailwind.config.js`, not copied gray-scale classes).
- **Policy pages** — `pages/termeni-si-conditii.js`, `pages/politica-de-confidentialitate.js`, `pages/politica-cookie-uri.js` → `LegalSection` over `data/legal.js` (`termsContent`, `privacyContent`, `cookiesContent`, plus `legalEntity = { name, city }` — no CIF, it's a PFA).
- **Footer** — `components/Home/Footer.jsx` has the three legal links + `<CookieSettingsButton className="text-xs" />`. Note: `CookieSettingsButton`'s default color was changed to `text-primary/50` (not `text-secondary/70` like guan/megadoor) because this Footer's background is dark (`bg-secondary`) — check the actual call-site background before reusing another project's default styling.

## Other siblings (spunei, societyhostess, cozyrelax, sommelier, friginstal, emda, …)

Not yet mapped. Run Phase 0 discovery; state which existing shape (guan/createvelab vs megadoor/homespa) the project matches, or that it has none and needs scaffolding, before editing anything.
