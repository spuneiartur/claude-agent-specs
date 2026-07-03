---
name: abandoned-slack-promises
version: 0.5.1
description: >
  Find Slack messages where the current user promised to do something but never followed up.
  Use this skill whenever someone asks to find their abandoned promises, forgotten follow-ups,
  dropped conversations, or unfinished commitments in Slack. Also trigger when someone says
  things like "what did I forget to do", "find threads I ghosted", "promises I didn't keep",
  "conversations I abandoned", or "things I said I'd check but didn't". Works for any user
  with Slack MCP tools connected — no configuration needed.
---

# Find My Abandoned Slack Promises

## Update check

Before starting, check if a newer version of this skill is available.

1. **Detect environment and locate skill directory:**
   - If `~/.claude/skills/abandoned-slack-promises/` exists → **Claude Code**. Use that
     as the skill dir. Use the same dir as the writable state dir.
   - Else if `.claude/skills/abandoned-slack-promises/` exists → **Claude Code (project-local)**.
     Same — use it for both skill dir and state dir.
   - Else if `/mnt/skills/user/abandoned-slack-promises/` or `/mnt/skills/org/abandoned-slack-promises/` exists → **Claude.ai** (use whichever matches).
     Skill dir is the matching path (read-only).
     Writable state dir is `~/.cache/skills/abandoned-slack-promises/` — create it
     with `mkdir -p` if it does not exist.
   - If none of the above exist → skip this section entirely.

2. **Check throttle** — read `.last-update-check` from the **writable state
   dir**. If it contains today's date (`YYYY-MM-DD`), skip the rest of this
   section.

3. **Write today's date** to `.last-update-check` in the **writable state dir**
   immediately (before any network call) so this check runs at most once per day.

4. **Check Atlassian MCP availability.** If the Atlassian MCP is not connected,
   print: "⚠️ Update check skipped — Atlassian MCP is not available." Then
   continue normally with the skill.

5. **Fetch the latest version** — use the Atlassian MCP to fetch Confluence
   page `7731806833`
   (`https://taxify.atlassian.net/wiki/spaces/BILP/pages/7731806833/Skills`).
   Find the version shown next to `abandoned-slack-promises` (format: `vX.Y.Z`). If the
   fetch fails, print: "⚠️ Update check failed — could not reach Confluence."
   Then continue normally.

6. **Compare versions** — compare the remote version with this skill's
   `version` field in the frontmatter above numerically, component by component:
   MAJOR first, then MINOR, then PATCH (e.g. `1.2.0` > `1.1.9`).
   If the remote version is newer:

   Tell the user:
   > ⬆️ A newer version of `abandoned-slack-promises` is available (vREMOTE vs local vLOCAL).
   >
   > Download: https://taxify.atlassian.net/wiki/download/attachments/7731806833/abandoned-slack-promises.skill
   >
   > I can download and install it now — update?

   - **Yes (Claude Code):** download the `.skill` file from the URL above,
     run `unzip -o abandoned-slack-promises.skill -d <parent-of-skill-dir>/`, then tell
     the user: "Updated! Restart Claude Code to use the new version."
   - **Yes (Claude.ai):** the skill mount is read-only and cannot be updated
     automatically. Tell the user: "Running in Claude.ai — please ask your
     org admin to update the skill mount to vREMOTE."
   - **No:** continue with the current version.

   If versions match or local is newer, print: "✅ `abandoned-slack-promises` vLOCAL is up to date."
   Then continue.

## Manual update

If the user explicitly asks to update this skill (e.g., "update this skill",
"check for updates", "upgrade abandoned-slack-promises"):

1. **Check Atlassian MCP availability.** If not connected, tell the user:
   "To check for updates, enable the Atlassian MCP connector." Then stop.
2. If connected, run the update check above **unconditionally** — start at
   step 1, skip steps 2–3 (throttle), and continue from step 4.

This skill searches Slack for messages where you promised to take action but never followed through. It verifies each hit by reading the full thread, filtering out false positives (resolved issues, real-time debugging, self-directed notes), and presents only genuinely abandoned commitments.

## Why this matters

In fast-moving Slack workspaces, it's easy to say "I'll take a look" or "let me check" and then get pulled into something else. These forgotten promises erode trust over time — someone is still waiting, or an issue remains unresolved. This skill surfaces those blind spots so you can close the loop.

## Required tools

You need these Slack MCP tools:
- `slack_search_public_and_private` (or `slack_search_public`)
- `slack_read_thread`
- `slack_search_users`

If these aren't available, tell the user they need to connect a Slack MCP connector first.

## Step 0 — Setup

### 0a — Compute the date window

Before searching, determine the cutoff date based on the requested window:
- **Last week**: today minus 7 days
- **Last month**: today minus 30 days
- **Last 2 months**: today minus 60 days

Format it as `YYYY-MM-DD` and append `after:YYYY-MM-DD` to every query in Step 1.

If the user doesn't specify a window, default to **last 2 months**.

### 0b — Identify the user

Use `slack_search_users` with the user's name to find their Slack user ID. You'll use this as a `from:<@USER_ID>` filter in every search query.

The Slack connector does NOT automatically filter searches to the authenticated user — without the `from:` filter, results will include messages from everyone in the organization.

### 0c — Detect languages from profile

Use `slack_read_user_profile` with `include_locale: true` to fetch the user's full profile. Scan all text fields (display name, title, "What I do", any custom fields) for language names — look for words like "Russian", "Ukrainian", "Spanish", "German", etc., in any language or script (e.g. "Русский", "Українська", "Deutsch").

Build the active language set from what you find:
- **Languages explicitly listed in the profile** → include their query set in Step 1.
- **English not listed** → add it anyway (always search English).
- **No languages found anywhere in the profile** → use English only.

Do not guess languages from the user's name, locale code, or writing style — only use what is explicitly stated in the profile fields.

## Step 1 — Search

### English queries (always included)

Run these 15 queries **in parallel** via `slack_search_public_and_private` (with `sort: "timestamp"`, `sort_dir: "desc"`, `limit: 20`, `include_context: false`):

1. `from:<@USER_ID> "I'll take a look" after:CUTOFF_DATE`
2. `from:<@USER_ID> "will check" after:CUTOFF_DATE`
3. `from:<@USER_ID> "let me check" after:CUTOFF_DATE`
4. `from:<@USER_ID> "will look" after:CUTOFF_DATE`
5. `from:<@USER_ID> "I'll look into" after:CUTOFF_DATE`
6. `from:<@USER_ID> "will review" after:CUTOFF_DATE`
7. `from:<@USER_ID> "get back to you" after:CUTOFF_DATE`
8. `from:<@USER_ID> "will investigate" after:CUTOFF_DATE`
9. `from:<@USER_ID> "will dig" after:CUTOFF_DATE`
10. `from:<@USER_ID> "let me investigate" after:CUTOFF_DATE`
11. `from:<@USER_ID> "I'll do" after:CUTOFF_DATE`
12. `from:<@USER_ID> "will do" after:CUTOFF_DATE`
13. `from:<@USER_ID> "I'll handle" after:CUTOFF_DATE`
14. `from:<@USER_ID> "will handle" after:CUTOFF_DATE`
15. `from:<@USER_ID> "I'll finish" after:CUTOFF_DATE`

### Additional language queries (run for languages found in the profile in Step 0c)

Append `after:CUTOFF_DATE` to all queries below as well.

**Russian:**
1. `from:<@USER_ID> "посмотрю"` (I'll look)
2. `from:<@USER_ID> "проверю"` (I'll check)
3. `from:<@USER_ID> "гляну"` (I'll glance/look)
4. `from:<@USER_ID> "разберусь"` (I'll figure it out)
5. `from:<@USER_ID> "вернусь к"` (I'll get back to)
6. `from:<@USER_ID> "сделаю"` (I'll do)
7. `from:<@USER_ID> "займусь"` (I'll handle)
8. `from:<@USER_ID> "закончу"` (I'll finish)

**Spanish:**
1. `from:<@USER_ID> "voy a revisar"` (I'll review)
2. `from:<@USER_ID> "lo reviso"` (I'll check it)
3. `from:<@USER_ID> "le echo un vistazo"` (I'll take a look)
4. `from:<@USER_ID> "lo verifico"` (I'll verify it)
5. `from:<@USER_ID> "lo hago"` (I'll do it)
6. `from:<@USER_ID> "me encargo"` (I'll handle it)
7. `from:<@USER_ID> "lo termino"` (I'll finish it)

**German:**
1. `from:<@USER_ID> "schaue ich mir an"` (I'll take a look)
2. `from:<@USER_ID> "prüfe ich"` (I'll check)
3. `from:<@USER_ID> "kümmere mich"` (I'll take care of it)
4. `from:<@USER_ID> "mache ich"` (I'll do it)
5. `from:<@USER_ID> "erledige ich"` (I'll handle it)
6. `from:<@USER_ID> "mache ich fertig"` (I'll finish it)

**Portuguese:**
1. `from:<@USER_ID> "vou verificar"` (I'll check)
2. `from:<@USER_ID> "vou dar uma olhada"` (I'll take a look)
3. `from:<@USER_ID> "vou investigar"` (I'll investigate)
4. `from:<@USER_ID> "vou fazer"` (I'll do it)
5. `from:<@USER_ID> "vou tratar"` (I'll handle it)
6. `from:<@USER_ID> "vou terminar"` (I'll finish it)

**French:**
1. `from:<@USER_ID> "je vais vérifier"` (I'll check)
2. `from:<@USER_ID> "je vais regarder"` (I'll look)
3. `from:<@USER_ID> "je regarde"` (I'll look into it)
4. `from:<@USER_ID> "je vais le faire"` (I'll do it)
5. `from:<@USER_ID> "je m'en occupe"` (I'll handle it)
6. `from:<@USER_ID> "je vais finir"` (I'll finish it)

**Polish:**
1. `from:<@USER_ID> "sprawdzę"` (I'll check)
2. `from:<@USER_ID> "zerknę"` (I'll glance)
3. `from:<@USER_ID> "ogarnę"` (I'll handle it)
4. `from:<@USER_ID> "zrobię"` (I'll do it)
5. `from:<@USER_ID> "zajmę się"` (I'll handle it)
6. `from:<@USER_ID> "skończę"` (I'll finish it)

**Turkish:**
1. `from:<@USER_ID> "bakacağım"` (I'll look)
2. `from:<@USER_ID> "kontrol edeceğim"` (I'll check)
3. `from:<@USER_ID> "yapacağım"` (I'll do it)
4. `from:<@USER_ID> "hallederim"` (I'll handle it)
5. `from:<@USER_ID> "bitireceğim"` (I'll finish it)

**Estonian:**
1. `from:<@USER_ID> "vaatan"` (I'll look)
2. `from:<@USER_ID> "kontrollin"` (I'll check)
3. `from:<@USER_ID> "uurin"` (I'll investigate)
4. `from:<@USER_ID> "tulen tagasi"` (I'll get back to)
5. `from:<@USER_ID> "teen ära"` (I'll do it)
6. `from:<@USER_ID> "võtan ette"` (I'll handle it)
7. `from:<@USER_ID> "lõpetan"` (I'll finish it)

**Finnish:**
1. `from:<@USER_ID> "katson"` (I'll look)
2. `from:<@USER_ID> "tarkistan"` (I'll check)
3. `from:<@USER_ID> "selvitän"` (I'll investigate/figure out)
4. `from:<@USER_ID> "palaan asiaan"` (I'll get back to it)
5. `from:<@USER_ID> "teen sen"` (I'll do it)
6. `from:<@USER_ID> "hoidan"` (I'll handle it)
7. `from:<@USER_ID> "viimeistelen"` (I'll finish it)

**Ukrainian:**
1. `from:<@USER_ID> "подивлюсь"` (I'll look)
2. `from:<@USER_ID> "перевірю"` (I'll check)
3. `from:<@USER_ID> "гляну"` (I'll glance/look)
4. `from:<@USER_ID> "розберусь"` (I'll figure it out)
5. `from:<@USER_ID> "повернусь до"` (I'll get back to)
6. `from:<@USER_ID> "зроблю"` (I'll do it)
7. `from:<@USER_ID> "займусь"` (I'll handle it)
8. `from:<@USER_ID> "закінчу"` (I'll finish it)

**Latvian:**
1. `from:<@USER_ID> "paskatīšos"` (I'll look)
2. `from:<@USER_ID> "pārbaudīšu"` (I'll check)
3. `from:<@USER_ID> "izpētīšu"` (I'll investigate)
4. `from:<@USER_ID> "atgriezīšos pie"` (I'll get back to)
5. `from:<@USER_ID> "izdarīšu"` (I'll do it)
6. `from:<@USER_ID> "nokārtošu"` (I'll handle it)
7. `from:<@USER_ID> "pabeigšu"` (I'll finish it)

**Lithuanian:**
1. `from:<@USER_ID> "pažiūrėsiu"` (I'll look)
2. `from:<@USER_ID> "patikrinsiu"` (I'll check)
3. `from:<@USER_ID> "išsiaiškinsiu"` (I'll figure it out)
4. `from:<@USER_ID> "grįšiu prie"` (I'll get back to)
5. `from:<@USER_ID> "padarysiu"` (I'll do it)
6. `from:<@USER_ID> "pasirūpinsiu"` (I'll handle it)
7. `from:<@USER_ID> "baigsiu"` (I'll finish it)

**Czech:**
1. `from:<@USER_ID> "podívám se"` (I'll look)
2. `from:<@USER_ID> "zkontroluju"` (I'll check)
3. `from:<@USER_ID> "prověřím"` (I'll investigate)
4. `from:<@USER_ID> "mrknu na to"` (I'll glance at it)
5. `from:<@USER_ID> "udělám to"` (I'll do it)
6. `from:<@USER_ID> "postarám se"` (I'll handle it)
7. `from:<@USER_ID> "dodělám"` (I'll finish it)

**Hungarian:**
1. `from:<@USER_ID> "megnézem"` (I'll look)
2. `from:<@USER_ID> "ellenőrzöm"` (I'll check)
3. `from:<@USER_ID> "utánanézek"` (I'll look into it)
4. `from:<@USER_ID> "visszatérek rá"` (I'll get back to it)
5. `from:<@USER_ID> "megcsinálom"` (I'll do it)
6. `from:<@USER_ID> "intézem"` (I'll handle it)
7. `from:<@USER_ID> "befejezem"` (I'll finish it)

**Romanian:**
1. `from:<@USER_ID> "mă uit"` (I'll look)
2. `from:<@USER_ID> "verific"` (I'll check)
3. `from:<@USER_ID> "investighez"` (I'll investigate)
4. `from:<@USER_ID> "revin cu"` (I'll get back with)
5. `from:<@USER_ID> "o fac"` (I'll do it)
6. `from:<@USER_ID> "mă ocup"` (I'll handle it)
7. `from:<@USER_ID> "termin"` (I'll finish it)

**Croatian:**
1. `from:<@USER_ID> "pogledat ću"` (I'll look)
2. `from:<@USER_ID> "provjerit ću"` (I'll check)
3. `from:<@USER_ID> "istražit ću"` (I'll investigate)
4. `from:<@USER_ID> "javim se"` (I'll get back to you)
5. `from:<@USER_ID> "napravit ću"` (I'll do it)
6. `from:<@USER_ID> "riješit ću"` (I'll handle it)
7. `from:<@USER_ID> "završit ću"` (I'll finish it)

**Serbian:**
1. `from:<@USER_ID> "погледаћу"` (I'll look)
2. `from:<@USER_ID> "проверићу"` (I'll check)
3. `from:<@USER_ID> "истражићу"` (I'll investigate)
4. `from:<@USER_ID> "јавим се"` (I'll get back to you)
5. `from:<@USER_ID> "урадићу"` (I'll do it)
6. `from:<@USER_ID> "решићу"` (I'll handle it)
7. `from:<@USER_ID> "завршићу"` (I'll finish it)

**Dutch:**
1. `from:<@USER_ID> "ik zal kijken"` (I'll look)
2. `from:<@USER_ID> "ik check het"` (I'll check it)
3. `from:<@USER_ID> "ik kijk ernaar"` (I'll look into it)
4. `from:<@USER_ID> "kom ik op terug"` (I'll get back to it)
5. `from:<@USER_ID> "ik doe het"` (I'll do it)
6. `from:<@USER_ID> "ik regel het"` (I'll handle it)
7. `from:<@USER_ID> "ik maak het af"` (I'll finish it)

**Italian:**
1. `from:<@USER_ID> "ci dò un'occhiata"` (I'll take a look)
2. `from:<@USER_ID> "controllo"` (I'll check)
3. `from:<@USER_ID> "verifico"` (I'll verify)
4. `from:<@USER_ID> "ci torno su"` (I'll get back to it)
5. `from:<@USER_ID> "lo faccio"` (I'll do it)
6. `from:<@USER_ID> "me ne occupo"` (I'll handle it)
7. `from:<@USER_ID> "finisco"` (I'll finish it)

**Swedish:**
1. `from:<@USER_ID> "ska kolla"` (I'll check)
2. `from:<@USER_ID> "ska titta"` (I'll look)
3. `from:<@USER_ID> "undersöker"` (I'll investigate)
4. `from:<@USER_ID> "återkommer"` (I'll get back)
5. `from:<@USER_ID> "ska göra"` (I'll do it)
6. `from:<@USER_ID> "tar hand om"` (I'll handle it)
7. `from:<@USER_ID> "ska slutföra"` (I'll finish it)

**Norwegian:**
1. `from:<@USER_ID> "skal sjekke"` (I'll check)
2. `from:<@USER_ID> "skal se på"` (I'll look at)
3. `from:<@USER_ID> "undersøker"` (I'll investigate)
4. `from:<@USER_ID> "kommer tilbake"` (I'll get back)
5. `from:<@USER_ID> "skal gjøre"` (I'll do it)
6. `from:<@USER_ID> "tar meg av"` (I'll handle it)
7. `from:<@USER_ID> "skal fullføre"` (I'll finish it)

**Swahili:**
1. `from:<@USER_ID> "nitaangalia"` (I'll look)
2. `from:<@USER_ID> "nitachunguza"` (I'll investigate)
3. `from:<@USER_ID> "nitakagua"` (I'll check)
4. `from:<@USER_ID> "nitarudi"` (I'll get back)
5. `from:<@USER_ID> "nitafanya"` (I'll do it)
6. `from:<@USER_ID> "nitashughulikia"` (I'll handle it)
7. `from:<@USER_ID> "nitamaliza"` (I'll finish it)

For languages not listed here, use your knowledge to generate 3-5 equivalent promise phrases in that language. The key patterns are: "I'll look", "I'll check", "I'll investigate", "I'll get back to you".

If any query returns the maximum number of results (20), paginate with the returned cursor to get more. The goal is comprehensive coverage.

## Step 2 — Verify each hit

For every message found, read the full thread with `slack_read_thread` (use `response_format: "concise"` to save tokens).

Mark a message as **ABANDONED** only if **all** of these are true:
- The user made a clear promise or commitment to act (not just discussing behavior of code)
- No substantive follow-up from the user exists later in the thread
- The topic was not resolved by someone else (no fix PRs, tickets closed, or confirmations)
- The requester did not self-resolve or indicate "all good" / "no worries" / "never mind"
- The thread is not a bot or automated message

**Exclude** a thread if ANY of these are true:
- Someone (the user or others) posted a fix, PR, ticket, deploy, or resolution
- The requester confirmed the issue is resolved or no longer needed
- It's a self-directed note with no one waiting on the user
- The conversation was real-time debugging that concluded in the same session (e.g., "let me check" followed by immediate investigation in the same thread)
- Any "done" signal exists: text ("fixed", "deployed", "merged", "resolved", "closed", "shipped", "released", "done", "solved") or emoji reactions (`:doneok:`, `:white_check_mark:`, `:heavy_check_mark:`, `:done:`, `:completed:`, `:solved:`, thumbs up from the requester)

This verification step is critical — in testing, ~95% of "promise" messages turn out to be followed through. Don't skip it.

### Step 2b — Reaction check (important!)

`slack_read_thread` does NOT return emoji reactions. So you can't see `:doneok:` or `:solved:` by reading the thread.

**Workaround:** Slack search supports `has::emoji:` which finds messages that have a specific reaction.

**Critical: use `after:/before:` date range, NOT `on:`.**
Slack search silently drops results when `from:` + `has::emoji:` + `on:` are combined. Instead, use a ±1 day range with `after:` and `before:` which is reliable under heavy filter stacking.

For each abandoned candidate, check exactly **two messages**:
1. **The user's promise message** — did anyone react with a done-emoji on it?
2. **The parent message** (thread starter) — did the user react with a done-emoji on it?

If either has a done-emoji → exclude the thread.

```
# 1. Check reactions on user's promise message (posted on 2023-07-14)
from:<@USER_ID> has::doneok: after:2023-07-13 before:2023-07-15 in:<#CHANNEL_ID>
# → if any result's message_ts matches the promise message → exclude
 
# 2. Check if user reacted on the parent message (posted on 2023-07-10)
hasmy::doneok: after:2023-07-09 before:2023-07-11 in:<#CHANNEL_ID>
# → if any result's thread_ts matches → exclude
```

Done-emojis to check: `:doneok:`, `:solved:`, `:white_check_mark:`

For DM channels, use `in:<@OTHER_USER_ID>` or the DM channel ID.

### Step 2c — DM fallback for standalone messages

Some DM messages are standalone (not in a thread). `slack_read_thread` won't show surrounding context for these. Use `slack_read_channel` with a timestamp range to see what was said before and after the promise message. Without this, DM candidates can't be properly verified.

## Step 3 — Output

Present results as a table, sorted by date **descending** (newest first):

| # | Priority | Date | Channel | Promise snippet | Thread link |
|---|----------|------|---------|-----------------|-------------|

Use emoji for priority:
- 🔴 **HIGH** — unresolved customer-facing or production issue, or an explicit ask from someone still waiting
- 🟡 **MEDIUM** — technical discussion or review request left hanging
- 🔵 **LOW** — casual, informational, or self-directed with low stakes

Every row **must** include a clickable thread link (permalink from the search results).

End with a summary line:

> Found **X** abandoned promises out of **Y** messages reviewed.

If nothing is found, say so clearly — a clean record is a good result. Do not fabricate or inflate results.
