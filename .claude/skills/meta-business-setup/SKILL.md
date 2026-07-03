---
name: meta-business-setup
version: 0.1.0
description: >
  Guide for setting up a new Meta (Facebook/Instagram) integration app from scratch —
  creating a dedicated app in Meta for Developers, generating access tokens via Graph
  API Explorer, granting ads_read / leads_retrieval permissions, connecting the app
  across Business Portfolios (agency app + client business), and creating a long-lived
  System User token for automation (crons, backend tools). Use whenever setting up a
  new Meta Marketing API or Lead Ads integration, connecting a new client's ad account
  or Page, or when asked "how do I set up Meta API access", "generate a Meta token",
  "connect a new client's ads account", "read leads from Meta", or "why do I need a
  separate app for this client". Captures the reasoning (why separate apps, why System
  Users) not just the click-path, since Meta's UI changes and the underlying model
  (Business Portfolio, app-to-business connection, asset-level roles) is what matters.
---

# Meta Business Setup — Ads + Leads Integration

This documents the full setup Artur did on 2026-07-03 to give an internal tool
(`spunei-api`) read access to a client's Meta Ads account and Lead Ads data, without
touching the client's production WhatsApp/messaging app. Follow this whenever
repeating the setup for a new client or a new capability (e.g. `ads_management`).

## Core principle: one app per purpose, not per client

A Meta "App" is just a container of permissions + credentials (App ID + App Secret).
Access tokens generated under it inherit whatever scopes the app has been granted.

**Never reuse a client's production integration app** (e.g. a WhatsApp Business
Messaging app with live system users wired to real traffic) for a new, unrelated
capability like ads reporting. Reasons:
- Blast radius: a bad token/bug in the new capability shouldn't be able to touch
  production messaging infrastructure.
- App Review risk: requesting new permissions can trigger review that temporarily
  restricts the *entire* app, including what's already live.
- Clean audit trail: separate apps = separate activity logs, easy to tell "was this
  API spike from ads reporting or from client messaging."

Instead: create **one small, purpose-built app per capability** (e.g. "Spunei Ads
Reporting" for Marketing API + Lead Ads), owned by your own Business Portfolio — not
the client's — because it's a reusable tool, not client property. It can still read
client assets (see below).

## Step 1 — Check whether an existing app is actually unused before reusing/deleting

Before creating anything, if there's ambiguity about which existing app is "the
production one" vs a leftover test app:

1. developers.facebook.com/apps → select the app → **Cazuri de utilizare / activity
   log** (left sidebar) — shows creation date, products added, Business Manager
   link/unlink history.
2. business.facebook.com/settings → **Conturi → Aplicații** → select the app →
   **Persoane** tab — if it has system users like `whatsapp-server` or
   `Conversions API System User` with full access, it's live production
   infrastructure. Don't touch it.
3. `grep -r "<APP_ID>"` across the project's codebase/`.env` files — if the App ID
   isn't referenced anywhere, it's safe to delete.

Only delete after confirming zero code references AND no live system users attached.

## Step 2 — Create the dedicated app

1. developers.facebook.com/apps → **Create New App**
2. Type: **Business**
3. Name it after the capability, not the client (e.g. "Spunei Ads Reporting", not
   "Guan Ads") — it'll be reused across clients later.
4. Use case: pick the narrowest one that unlocks what you need — e.g. **"Create &
   manage ads with Marketing API"** for `ads_read`. Don't add unrelated use cases
   (WhatsApp, Facebook Login, Threads) just because they're offered.
5. Business Portfolio: connect it to **your own** portfolio (agency), not the
   client's — create one here if you don't have one yet ("Create a business
   portfolio"). This doesn't limit which ad accounts/pages the app can later read —
   that's governed by asset-level roles, not app ownership.

## Step 3 — Quick test access via Graph API Explorer (Standard Access, no App Review)

For reading **your own or an admin'd client's** ad account/page — i.e. any asset
where your personal Facebook account already has a role — you do **not** need Meta
App Review / Advanced Access. This is called Standard Access and works indefinitely.
App Review is only required when the app needs to access assets belonging to people
who haven't granted you a role directly.

1. developers.facebook.com/tools/explorer
2. **Meta App**: select your new app
3. **User or Page**: "Get User Access Token" (for ad account / insights access)
4. **Permissions** → add `ads_read` → **Generate Access Token** → approve in the
   popup
5. Copy the token — **do not paste it into a chat/conversation log**, even a
   short-lived one. Put it directly into the target `.env` file yourself, or hand it
   to the assistant only if it's a genuinely temporary (~1h) test token and you
   accept it'll sit in that conversation's history.
6. Ad Account ID: business.facebook.com/settings → **Conturi → Conturi de
   publicitate** → select the account → format as `act_<ID>` in `.env`.

This is enough for a one-off pull (e.g. `meta-fetch.py` creative/performance export).

## Step 4 — Lead Ads read access needs a **Page** token, not a User token

Meta's Leads API (`/{page_id}/leadgen_forms`, `/{form_id}/leads`) structurally
rejects User Access Tokens regardless of scopes granted on them. It requires a Page
Access Token, and getting there has two separate gotchas — hitting either one alone
still leaves you stuck, so do both in order.

### Gotcha 1 — the Page belongs to a different Business Portfolio than your app

If `/me/accounts?fields=id,name,access_token&access_token=<USER_TOKEN>` returns an
**empty list** despite you having full admin access on the Page, it's because the
Page belongs to the **client's** Business Portfolio while your app belongs to
**your own** (by design, per Step 2). Personal admin access on the Page doesn't
automatically extend to a foreign app's tokens.

**Fix — connect your app to the client's Business Portfolio:**

1. business.facebook.com/settings → switch to the **client's** portfolio (top
   selector)
2. **Conturi → Aplicații** → **+ Adaugă** → **"Ce dorești să faci?"** dialog appears
   — choose **"Conectează un ID de aplicație"** (NOT "Creează un nou ID de
   aplicație" and NOT "Solicită acces la un ID de aplicație" — the app is already
   yours, it just needs linking).
3. Paste your app's App ID (found in developers.facebook.com/apps → App settings →
   Informații generale — this is a public identifier, safe to share/paste anywhere).
4. Confirm — the app now shows in that business's Apps list, labeled "Activ deținut
   de \<Your Agency\>" — ownership stays with you, access is just extended.

Once connected, the client's Pages/ad accounts appear **directly in Graph API
Explorer's "User or Page" dropdown** by name — you no longer need the `/me/accounts`
workaround at all for manual testing (it's still useful for scripted/automated
token derivation later).

### Gotcha 2 — `leads_retrieval` / `pages_manage_ads` don't appear in Explorer's permission search at all

This is a separate problem from Gotcha 1, and fixing Gotcha 1 doesn't fix this one.
Some permissions (leads-related ones especially) are gated behind a **use case**
that hasn't been added to the app yet — searching for them in Explorer's "Add a
Permission" box returns nothing until the app itself declares that use case.

**Fix — add the use case first, at the app level:**

1. developers.facebook.com/apps → your app → **Cazuri de utilizare** (left sidebar)
2. **Add use cases** (top right)
3. Filter by **"Ads and monetization"**
4. Check **"Capture & manage ad leads with Marketing API"** (this is what unlocks
   `leads_retrieval`; it's a distinct use case from "Create & manage ads with
   Marketing API" added in Step 2 — you'll end up with both)
5. **Save**

Only after this does `leads_retrieval` (and `pages_manage_ads`, which the Leads API
also demands) show up as searchable/grantable in Graph API Explorer.

### Putting it together

- Permissions needed for leads: `leads_retrieval`, `pages_manage_ads`,
  `pages_show_list`, `pages_read_engagement`.
- Always verify what actually got granted after generating a token — Explorer
  sometimes silently drops a checked permission, especially sensitive ones:
  ```
  GET /me/permissions?access_token=<TOKEN>
  ```
  If a needed permission shows missing, redo the OAuth popup using **"Editează
  setările"** (not the quick "Reconectează-te") to reach the explicit
  permission-confirmation screen.

## Step 5 — System User token for automation (long-lived, not tied to your login)

For anything that runs unattended (a cron job, a backend tool), don't rely on a
personal User/Page token — those expire and break if you change your password or
lose the session. Use a System User instead:

1. Client's Business Settings → **Utilizatori → Utilizatori de sistem** → **+
   Adaugă** → name it (e.g. `spunei-ads-integration`) → role **Admin**
2. Select the System User → **Atribuie active** → grant **Control total** on the
   Page and/or Ad Account it needs to read
3. **Generează un nou token** → select your app (now connected per Step 4) → tick
   the permissions needed (`ads_read`, `leads_retrieval`, `pages_manage_ads`,
   `pages_read_engagement`)
4. Set expiration to **never**, if offered

Store this token as the long-lived credential for the cron/tool. Nothing about it
depends on any individual's personal Facebook session.

## What the Marketing API can and can't extract (confirmed 2026-07-03)

Verified live against a real account (`ads_read`/`ads_management` scopes, no extra
permissions needed beyond Step 3/4):

| Data | Field / endpoint | Status |
|---|---|---|
| Ad copy (primary text, headline, description, CTA) | `creative{body,title,object_story_spec}` | ✅ Confirmed — see `meta-fetch.py` |
| Multiple dynamic text/headline variants | `creative.asset_feed_spec` | ✅ Confirmed |
| Static image URL | `creative.image_url` / `thumbnail_url` | ✅ Confirmed |
| Video metadata (duration, permalink) | `GET /{video_id}?fields=length,permalink_url` | ✅ Confirmed — caught a real 35.92s video running as a Reel (15s max recommended) |
| Video file itself (downloadable) | `GET /{video_id}?fields=source` | ❌ Not returned for ads-delivered video (likely rights/licensing restriction) — works for Page-posted videos, not ad creatives |
| Learning phase status per ad set | `GET /{adset_id}?fields=learning_stage_info` | ✅ Confirmed — returns `status: LEARNING \| LEARNING_LIMITED \| SUCCESS \| FAIL` + `conversions` count. Matches the "Învățare limitată" badge in Ads Manager UI exactly. |
| Meta's own optimization recommendations | `GET /{ad_account_id}/recommendations` | ✅ Confirmed, and genuinely high-value — returned a `FRAGMENTATION` recommendation naming two specific overlapping ad sets, matching a finding we'd independently spotted manually. Also flags conversion-optimization and creative (auto-music) opportunities. No extra permission required. |
| Campaign/ad set structure, budget, bid strategy, targeting spec | `campaigns`/`adsets` fields (`daily_budget`, `bid_strategy`, `targeting`, `optimization_goal`) | ✅ Confirmed |
| Pixel health (last fired time, setup) | `{ad_account_id}/adspixels` | ✅ Confirmed |

**Takeaway for the future `meta-ads.js` tool**: `learning_stage_info` and
`/recommendations` are the highest-value fields to include — they surface Meta's
own diagnosis of budget fragmentation and optimization issues without any manual
audit work. Both work with the same `ads_read`/`ads_management` scopes already
covered in Step 3, no incremental permission cost.

## Token handling hygiene

- Public identifiers (App ID, Ad Account ID, Page ID) are fine to paste anywhere,
  including chat — they're not secrets.
- Access tokens are secrets. Prefer editing target `.env` files directly over
  pasting tokens into a conversation. A short-lived (~1h) Explorer test token is low
  risk if pasted, but the habit matters more once long-lived/System User tokens are
  involved — those must never be pasted into chat or committed to git.
