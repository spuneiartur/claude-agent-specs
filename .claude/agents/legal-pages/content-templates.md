# Content templates — Terms / Privacy / Cookies

Section skeletons proven across guan/megadoor/homespa/createvelab. Adapt wording to the project's tone and language — don't paste verbatim. Every `{legalEntity...}` placeholder must come from real data (constitution rule #1) — never fabricate.

Data shape assumed: `legalEntity = { name, city, cif?, tradeRegister?, address? }`. Fields that don't apply (e.g. `cif` for a PFA with none) are simply omitted from the rendered list — don't print an empty "CIF: " line.

## Terms & Conditions — section order

1. **Introduction** — welcomes the visitor, states that using the site implies acceptance of these terms.
2. **Company/Operator Information** — bullet list: name, registered office/city, trade register number (if any), CIF (if any). Omit fields the entity doesn't have.
3. **Description of Services** — one paragraph on what the site/business actually offers (pull from the homepage's own description — don't generalize into "various services").
4. **Intellectual Property Rights** — entity owns all site content (text/images/video/logo/code); visitor may not copy/redistribute for commercial use; personal non-commercial printing/downloading is fine.
5. **Limitation of Liability** — no guarantee of uninterrupted/error-free operation or absence of harmful components; no liability for damages from use/inability to use the site.
6. **Third-Party Links** — site may link to third-party platforms (Instagram, WhatsApp, etc.); entity doesn't control or take responsibility for them.
7. **Governing Law and Jurisdiction** — Romanian law + Romanian courts for RO-registered entities; adapt for other jurisdictions.

## Privacy Policy — section order

1. **Introduction** — GDPR (EU) 2016/679 commitment statement, one paragraph.
2. **Data Controller** — entity name + city/address (this is the one section that's structurally mandatory even for a PFA with no CIF — GDPR requires an identifiable controller).
3. **What Personal Data We Collect** — bulleted by category: Identity & Contact (name/email/phone — only if the site actually collects these, e.g. via a contact form or booking flow), Technical (IP/browser/device — only collected via cookies if analytics consent given), Usage Data.
4. **Purpose and Legal Basis for Processing** — bulleted: responding to inquiries (legitimate interest/consent), improving the site via analytics (consent via cookies). Don't list purposes the site doesn't actually have (e.g. no "processing payments" bullet if there's no checkout).
5. **Data Sharing and Transfer** — no selling of data; may share with trusted processors (hosting, analytics providers) bound by GDPR.
6. **Data Retention** — retained only as long as necessary for the stated purposes / legal obligations.
7. **Your GDPR Rights** — access, rectification, erasure, restriction, portability, objection, withdraw consent — plus, for Romanian entities, the right to complain to **ANSPDCP** (Autoritatea Națională de Supraveghere a Prelucrării Datelor cu Caracter Personal).

## Cookie Policy — section order

1. **What Are Cookies** — generic, one paragraph, doesn't need entity-specific wording.
2. **How We Use Cookies** — entity-specific: what the site uses them for.
3. **Categories** — mirror `COOKIE_CATEGORIES` exactly, one bullet/subsection per category (`necessary`, `analytics`, `functional` if present). **Cross-check against the actual config file before writing this — do not describe a category or vendor that isn't in `COOKIE_CATEGORIES`.**
4. **Third-Party Cookies** — enumerate only vendors actually wired (`grep` for `GTM_ID`/`GA4_ID`/`PIXEL_ID`/`CLARITY_ID` etc. per Phase 0 step 7). If none: say so explicitly — "Momentan nu folosim niciun instrument de analiză sau publicitate terț pe acest site. Această secțiune va fi actualizată de imediat ce adăugăm unul." (or the English equivalent) — never leave the topic silently unaddressed.
5. **How to Manage Your Cookie Preferences** — points to the consent banner + a footer "cookie settings" link/button (`CookieSettingsButton`) that calls `resetConsent()`.

## Seam with `tracking-integrator`

When `tracking-integrator` wires a new vendor tag, its own constitution rule #4 requires it to update the privacy/cookie disclosure using the vendor's *official* disclosure doc. In practice that means: it adds the cookie entry to `COOKIE_CATEGORIES` (`cookie-config.js`) and extends step 3/4 above in the relevant policy content file. `legal-pages` doesn't need to re-do that work — but if asked to audit a project's legal pages, check `COOKIE_CATEGORIES` against the policy text and flag/fix any drift (a vendor present in code but not disclosed, or vice versa).

## Placeholder convention (when legal identity is genuinely unknown)

If Artur explicitly asks to proceed without the real entity data (rare — usually you should just ask, per constitution rule #1), use an unmissable bracketed placeholder, not a plausible-looking fake:

```
[COMPLETEAZĂ: nume operator]
[COMPLETEAZĂ: CIF]
[COMPLETEAZĂ: adresă/localitate]
```

Never write something that reads as a real company (no "Exemplu S.R.L.", no invented CIF format like "RO12345678") — the risk is Artur forgets to replace it and ships fiction as a legal disclosure.
