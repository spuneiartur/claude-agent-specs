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
Access Token.

- Permissions needed on top of the above: `leads_retrieval`, `pages_show_list`,
  `pages_read_engagement`.
- **Gotcha**: `leads_retrieval` sometimes doesn't get granted even if you tick it in
  Explorer — always verify after generating with:
  ```
  GET /me/permissions?access_token=<TOKEN>
  ```
  and confirm `leads_retrieval` shows `"status": "granted"`. If missing, redo the
  OAuth popup and use **"Editează setările"** (not the quick "Reconectează-te") to
  reach the explicit Page-selection screen.
- You can derive a Page token from an already-authorized User token instead of
  fighting the Explorer's "User or Page" dropdown:
  ```
  GET /me/accounts?fields=id,name,access_token&access_token=<USER_TOKEN>
  ```
  Each Page you administer comes back with its own `access_token`.

### If `/me/accounts` returns an empty list despite having Page admin access

This happens when the Page belongs to a **different Business Portfolio** than the
one your app is connected to (e.g. Page owned by "Client Business", app owned by
"Your Agency"). Personal admin access on the Page doesn't automatically extend to a
foreign app's tokens. Fix: connect the app to the client's business portfolio (next
step) — this is a formal, one-time link, separate from your personal Page role.

## Step 5 — Connect your app to the client's Business Portfolio

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

## Step 6 — System User token for automation (long-lived, not tied to your login)

For anything that runs unattended (a cron job, a backend tool), don't rely on a
personal User/Page token — those expire and break if you change your password or
lose the session. Use a System User instead:

1. Client's Business Settings → **Utilizatori → Utilizatori de sistem** → **+
   Adaugă** → name it (e.g. `spunei-ads-integration`) → role **Admin**
2. Select the System User → **Atribuie active** → grant **Control total** on the
   Page and/or Ad Account it needs to read
3. **Generează un nou token** → select your app (now connected per Step 5) → tick
   the permissions needed (`ads_read`, `leads_retrieval`, `pages_read_engagement`)
4. Set expiration to **never**, if offered

Store this token as the long-lived credential for the cron/tool. Nothing about it
depends on any individual's personal Facebook session.

## Token handling hygiene

- Public identifiers (App ID, Ad Account ID, Page ID) are fine to paste anywhere,
  including chat — they're not secrets.
- Access tokens are secrets. Prefer editing target `.env` files directly over
  pasting tokens into a conversation. A short-lived (~1h) Explorer test token is low
  risk if pasted, but the habit matters more once long-lived/System User tokens are
  involved — those must never be pasted into chat or committed to git.
