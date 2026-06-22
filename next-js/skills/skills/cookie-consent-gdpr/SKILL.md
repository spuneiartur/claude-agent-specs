---
name: cookie-consent-gdpr
description: >
  Wire third-party tracking, analytics, advertising, marketing pixels, or any cookie-setting
  script into the project's Cookie Consent + GDPR system. Use this skill whenever the user
  asks to add Google Tag Manager (GTM), Google Analytics (GA4), Meta/Facebook Pixel, TikTok
  Pixel, LinkedIn Insight, HotJar, Clarity, Hubspot, Intercom, marketing tags, conversion
  pixels, retargeting scripts, or any 3rd-party tracker. Also trigger when the user mentions
  "tag manager", "marketing pixel", "analytics", "tracking script", "consent mode", "GDPR
  compliance", "cookie banner integration", or shares a script snippet from a marketing/SEO
  agency to install on the site. Covers both the SSR Consent Mode v2 pattern (visible in HTML
  on every page) and the client-side gated pattern (loads after user accepts).
---

# Cookie Consent + GDPR Integration

This project has a built-in cookie consent system. Every script that sets cookies, fingerprints, tracks, or sends user data to a third party MUST respect it. There are two correct patterns — pick one based on the script's requirements.

## The two patterns

### Pattern A — SSR Consent Mode (Google Tag Manager, Google Ads, Google Analytics)

Use when the script supports Google **Consent Mode v2**. The tag is rendered server-side on every page (visible in "View Source") but receives a `consent: default = denied` signal at load. When the user accepts cookies, a `consent: update` signal is sent and tags fire.

**Why:** marketing/SEO agencies expect to see the tag in HTML on every page (their auditors check this). Consent Mode v2 keeps the site GDPR-compliant *while* making the tag visible.

**Reference implementation:** `components/GoogleTagManager.jsx` + `components/GoogleTagManagerNoscript.jsx` + `functions/update-gtm-consent.js`.

Files involved:
- `components/{Vendor}.jsx` — server-side component, renders `<script>` with consent default + loader. No hooks. Imported in `pages/_document.js` inside `<Head>`.
- `components/{Vendor}Noscript.jsx` — server-side component, renders `<noscript>` iframe. Imported in `pages/_document.js` immediately after `<body>`.
- `functions/update-{vendor}-consent.js` — pure function that calls `window.gtag('consent', 'update', {...})` (or vendor equivalent) with the consent state mapped to vendor categories.
- `contexts/CookieConsentContext.jsx` — already wired. The `saveConsent` function and the localStorage-load `useEffect` call `updateGtmConsent(consent)`. Add a similar call for any new vendor.

Environment variable: declare the ID as `{VENDOR}_ID` in `.env` (server-side only, no `NEXT_PUBLIC_` prefix — `_document.js` is SSR).

### Pattern B — Client-side gated (TikTok embed, YouTube embed, HotJar, Intercom, marketing pixels without consent mode)

Use when the script does NOT support consent mode, or when the script should not load at all without consent (heavier privacy stance).

**Reference implementation:** `components/TikTokEmbed.jsx` using `ConsentGate` from `@components/CookieConsent`.

Two sub-variants:

1. **Inline content gate (embed):** wrap the rendered content with `<ConsentGate category="functional" fallbackMessage="...">`. Renders a "please accept cookies" placeholder until consent.

2. **Background script gate:** in a client component, read `useCookieConsent()` and return `null` until `isAllowed('analytics')` (or the matching category) is true. When true, render the script via `next/script` with `strategy="afterInteractive"`. Mount the component in `pages/_app.js` inside the `CookieConsentProvider`.

Environment variable: must be `NEXT_PUBLIC_{VENDOR}_ID` because the component runs client-side.

## Cookie categories

The project has three categories defined in `contexts/CookieConsentContext.jsx`:

| Category     | What goes here                                                          |
|--------------|-------------------------------------------------------------------------|
| `necessary`  | Always on. Session, auth, CSRF. No script needs to opt-in.              |
| `functional` | Embeds (YouTube, TikTok), live chat (Intercom), preference storage.     |
| `analytics`  | GTM, GA4, Meta Pixel, TikTok Pixel, HotJar, Clarity, marketing pixels.  |

If a vendor doesn't fit, **do not invent a new category**. Talk to the user first — adding a category means updating the cookie banner UI in `components/CookieConsent/`.

## Decision tree

When the user shares a 3rd-party script:

1. **Does the vendor support Consent Mode v2?** (Google products: GTM, GA4, Google Ads → yes. Most others → no.)
   - Yes → Pattern A (SSR, visible in HTML).
   - No → Pattern B (client-side gated).

2. **Is the request from a marketing/SEO agency that explicitly said "must be in `<head>` on every page"?**
   - Yes → Pattern A is mandatory. If the vendor doesn't support consent mode, push back to the user — explain that loading without consent breaks GDPR.

3. **Is it an inline embed (YouTube, TikTok, social card)?**
   - Use Pattern B with `<ConsentGate>` for visible fallback UI.

4. **Is it a heavy background script (HotJar, FullStory, Intercom widget)?**
   - Use Pattern B without a fallback (null until consent).

## Pattern A — Step-by-step (Consent Mode v2 vendor)

1. Create `components/{Vendor}.jsx`:
   ```jsx
   const consentDefault = `window.dataLayer = window.dataLayer || [];
   function gtag(){dataLayer.push(arguments);}
   gtag('consent', 'default', {
     ad_storage: 'denied',
     ad_user_data: 'denied',
     ad_personalization: 'denied',
     analytics_storage: 'denied',
     functionality_storage: 'denied',
     personalization_storage: 'denied',
     security_storage: 'granted',
     wait_for_update: 500
   });`;

   const loader = (id) => `/* vendor inline loader using ${id} */`;

   const Vendor = () => {
     const id = process.env.VENDOR_ID;
     if (!id) return null;
     return (
       <>
         {/* Vendor - Consent Mode Default */}
         <script dangerouslySetInnerHTML={{ __html: consentDefault }} />
         <script dangerouslySetInnerHTML={{ __html: loader(id) }} />
         {/* End Vendor */}
       </>
     );
   };
   export default Vendor;
   ```
   Note: if the **previous** vendor in `_document.js` already set the consent default (e.g., GTM is there), do NOT duplicate the `consentDefault` block — only emit the vendor's own loader.

2. Create `components/{Vendor}Noscript.jsx` if the vendor needs a `<noscript>` fallback iframe (GTM does; GA4 doesn't).

3. Export both from `components/index.js` (alphabetical order).

4. Import in `pages/_document.js`:
   ```jsx
   <Head>
     <Vendor />
     <AppHead />
   </Head>
   <body>
     <VendorNoscript />
     <Main />
     <NextScript />
   </body>
   ```

5. Create `functions/update-{vendor}-consent.js`:
   ```js
   const update{Vendor}Consent = (consent) => {
     if (typeof window === 'undefined' || !window.gtag) return;
     const value = (allowed) => (allowed ? 'granted' : 'denied');
     window.gtag('consent', 'update', { /* map to vendor categories */ });
   };
   export default update{Vendor}Consent;
   ```
   Export from `functions/index.js`.

6. Wire the function into `contexts/CookieConsentContext.jsx`:
   - Call `update{Vendor}Consent(parsed)` inside the `useEffect` that loads from localStorage (after `setHasConsented(true)`).
   - Call `update{Vendor}Consent(consentToSave)` inside `saveConsent` (after `setHasConsented(true)`).

7. Add `VENDOR_ID=...` to `.env` and `.env.example` (no `NEXT_PUBLIC_` prefix).

8. Verify in the browser:
   - Open DevTools → Network. With no consent stored, the vendor's loader should fire but tags inside should NOT (Consent Mode blocks them).
   - Click "Accept" → tags should fire on the next event.
   - Right-click → "View Source" should show the tag in `<head>` on every page, including dynamic routes.

## Pattern B — Step-by-step (Client-side gated)

1. Create `components/{Vendor}.jsx`:
   ```jsx
   import { useCookieConsent } from 'contexts';
   import Script from 'next/script';

   const Vendor = () => {
     const { isAllowed } = useCookieConsent();
     const id = process.env.NEXT_PUBLIC_VENDOR_ID;
     if (!isAllowed('analytics') || !id) return null;
     return (
       <Script
         id="vendor-loader"
         strategy="afterInteractive"
         dangerouslySetInnerHTML={{ __html: `/* vendor loader using ${id} */` }}
       />
     );
   };
   export default Vendor;
   ```

2. Export from `components/index.js`.

3. Import in `pages/_app.js` **inside** `CookieConsentProvider` (so `useCookieConsent` works), **outside** `QueryClientProvider`:
   ```jsx
   <CookieConsentProvider>
     <Vendor />
     <QueryClientProvider client={queryClient}>
       <Component {...pageProps} />
     </QueryClientProvider>
     ...
   </CookieConsentProvider>
   ```

4. For inline embeds (YouTube, TikTok, social), use `<ConsentGate>` instead:
   ```jsx
   import { ConsentGate } from '@components/CookieConsent';

   const VendorEmbed = ({ url }) => (
     <ConsentGate
       category="functional"
       fallbackMessage="Pentru a vedea acest conținut, te rugăm să accepți cookie-urile funcționale."
     >
       <VendorEmbedContent url={url} />
     </ConsentGate>
   );
   ```

5. Add `NEXT_PUBLIC_VENDOR_ID=...` to `.env` and `.env.example`.

## Anti-patterns — do NOT do these

- ❌ Inserting a raw `<script>` directly into `pages/_document.js` without consent handling.
- ❌ Using Pattern A for a vendor that doesn't support consent mode (e.g., raw Meta Pixel without their Consent API).
- ❌ Creating a one-off `useEffect` in `_app.js` that conditionally appends a `<script>` to `document.head` — use the components above instead.
- ❌ Hardcoding tracking IDs in source code. Always use env vars.
- ❌ Adding a new cookie category silently. The banner UI must be updated to surface it to users.
- ❌ Forgetting to also update `CookieConsentContext` when adding a Pattern A vendor — the consent update will never fire and the tag will stay denied forever.
- ❌ Loading any tracker BEFORE the consent default is set. In Pattern A, `consentDefault` script must come FIRST in `<head>` before any vendor loader.

## Privacy policy & cookie policy

If the user installs a new vendor, prompt them to update the public cookie/privacy policy page (typically `pages/politica-cookies.js` or similar) with: vendor name, purpose, cookies set, retention. The banner UI also lists vendors in `components/CookieConsent/DetailedSettings.jsx` — check whether the new vendor needs to be added there.
