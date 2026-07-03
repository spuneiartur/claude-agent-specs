---
name: find-answer
version: 0.3.1
description: >
  Find the answer to a Slack question by searching that channel's own prior history.
  Given a Slack message link, search the channel for similar past threads, aggregate
  the recommendations or confirmed answers, and prepare a draft threaded reply with
  attributions and permalinks back to the source. Always drafts, never posts
  automatically — asks the user whether to post or just keep the text. Channel-agnostic.
  Use whenever the user asks 'find answer for this slack thread', 'find an answer in
  this channel', 'see if this was asked before', 'summarise what people recommended in
  this channel', or shares a Slack message link and wants a reply built from prior
  threads. Also trigger on 'this has been asked before, draft a reply' or 'pull past
  recommendations from the channel'. Handles two question shapes: recommendation
  gathering (bulleted summary) and narrow factual questions (single confident answer
  only if past threads agree).
---

# Find Answer (from Slack channel history)

## Update check

Before starting, check if a newer version of this skill is available.

1. **Detect environment and locate skill directory:**
   - If `~/.claude/skills/find-answer/` exists → **Claude Code**. Use that
     as the skill dir. Use the same dir as the writable state dir.
   - Else if `.claude/skills/find-answer/` exists → **Claude Code (project-local)**.
     Same — use it for both skill dir and state dir.
   - Else if `/mnt/skills/user/find-answer/` or `/mnt/skills/org/find-answer/` exists → **Claude.ai** (use whichever matches).
     Skill dir is the matching path (read-only).
     Writable state dir is `~/.cache/skills/find-answer/` — create it
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
   Find the version shown next to `find-answer` (format: `vX.Y.Z`). If the
   fetch fails, print: "⚠️ Update check failed — could not reach Confluence."
   Then continue normally.

6. **Compare versions** — compare the remote version with this skill's
   `version` field in the frontmatter above numerically, component by component:
   MAJOR first, then MINOR, then PATCH (e.g. `1.2.0` > `1.1.9`).
   If the remote version is newer:

   Tell the user:
   > ⬆️ A newer version of `find-answer` is available (vREMOTE vs local vLOCAL).
   >
   > Download: https://taxify.atlassian.net/wiki/download/attachments/7731806833/find-answer.skill
   >
   > I can download and install it now — update?

   - **Yes (Claude Code):** download the `.skill` file from the URL above,
     run `unzip -o find-answer.skill -d <parent-of-skill-dir>/`, then tell
     the user: "Updated! Restart Claude Code to use the new version."
   - **Yes (Claude.ai):** the skill mount is read-only and cannot be updated
     automatically. Tell the user: "Running in Claude.ai — please ask your
     org admin to update the skill mount to vREMOTE."
   - **No:** continue with the current version.

   If versions match or local is newer, print: "✅ `find-answer` vLOCAL is up to date."
   Then continue.

## Manual update

If the user explicitly asks to update this skill (e.g., "update this skill",
"check for updates", "upgrade find-answer"):

1. **Check Atlassian MCP availability.** If not connected, tell the user:
   "To check for updates, enable the Atlassian MCP connector." Then stop.
2. If connected, run the update check above **unconditionally** — start at
   step 1, skip steps 2–3 (throttle), and continue from step 4.

Use this skill when the user wants to answer a Slack question by reusing what the channel itself has already said. The skill assumes the *channel* contains the institutional knowledge — it does not invent answers, only aggregates and attributes.

## When to use

Trigger on requests like:
- "Find answer for this slack thread: SLACK_URL"
- "Reply to this thread with what people have said before in this channel"
- "Find similar questions in this channel and summarise the answers"
- "find-answer for this message: SLACK_URL"
- "Has this been asked before in this channel? If so, draft a reply"
Do NOT use this skill for:
- General web research (no Slack history involved)
- Questions where the channel is unlikely to contain the answer (e.g. asking about industry-wide knowledge in a private 5-person channel)
- For-sale posts, casual chat, or announcements
## Required tools

Slack MCP:
- `slack_search_public_and_private` (or `slack_search_public`)
- `slack_read_thread`
- `slack_read_channel` (optional, for context)
- `slack_send_message` and/or `slack_send_message_draft`
- `slack_search_channels` (only if the user gave a channel name and you need its ID)
- `slack_search_users` (to resolve display names to user IDs for correct `<@USER_ID>` mentions)
## Inputs the skill needs

Before searching, confirm you have:
1. **Channel ID** (e.g. `C09RRHCKR3J`). If only a channel name was given, resolve via `slack_search_channels`.
2. **Question message timestamp** (`message_ts`) — the parent message you'll reply to. Accept either `message_ts` directly or a Slack permalink `…/archives/<channel>/p<digits>` and convert by inserting a decimal: `p1778056897398289` → `1778056897.398289`.
3. The skill **always prepares a draft** and **never posts automatically**. After the draft is shown, the skill asks the user whether to post it as a thread reply or just leave it as text. Do not assume "post directly" mode even if the user's original prompt sounds eager — confirm explicitly after the draft is composed.
If channel ID or message timestamp are missing or ambiguous, ask the user before proceeding.

## Step 1 — Read the question

Call `slack_read_thread` with the parent `message_ts`. Extract:
- The question text (parent message body)
- The author's user ID (for the greeting in the reply)
- Existing replies — if there are already substantive replies (>40 chars, not pure emoji), tell the user "this thread already has answers" and ask whether to proceed anyway.
## Step 2 — Classify the question

Pick exactly one label:

- **`RECOMMENDATION_GATHERING`** — open-ended ask for opinions/options. Cues: "any tips on…", "what do you use for…", "where can I get…", "experiences with…", "recommend a…", "anyone knows a place…", "is there a good…".
- **`SIMPLE_QUESTION`** — narrow, factual, scoped to a single answer. Cues: "is X open today?", "does Y require Z?", "what's the limit for…", "is the API endpoint for X public?".
- **`OTHER`** — chat, announcement, for-sale, off-topic, opinion poll without a clear factual answer → tell the user "this isn't a knowledge-retrieval question" and stop.
## Step 3 — Search the channel for prior similar threads

Run AT MOST 3 `slack_search_public_and_private` calls. Each query MUST scope with `in:<#CHANNEL_ID>`. Build keyword variations from the question's core nouns and verbs — vary by 2–3 different angles (synonyms, brand vs category, problem vs solution).

Examples for "Where to inspect a Toyota before buying?" in #all-estonia-random:
- Query 1: `in:<#C…> pre-purchase inspection used car`
- Query 2: `in:<#C…> recommend auto shop check before buying`
- Query 3: `in:<#C…> Toyota inspection`
Set `limit=10`, `sort=score`. Use natural-language queries when the channel is small; switch to keyword-only when it's high-volume.

Filter the merged results:
- `Reply count >= 2` (the thread actually got engagement)
- Not the current question (`message_ts` mismatch)
- Not a for-sale / announcement post (look at the text — `for sale`, `:blue_car:`, model + year + price → drop)
- Within the last ~24 months unless the channel is slow-moving
Pick the top 3–5 most relevant threads. Read each with `slack_read_thread` (`response_format=concise`).

### Wide search (cross-channel)

If the channel-scoped search yields no useful results, the user may request a **wide search** across the whole workspace (dropping the `in:<#CHANNEL_ID>` scope). When doing so:

**Privacy protection — MANDATORY:**
- For every result returned, check the channel type. **Discard any result that comes from a private channel** (channel name prefixed with `🔒`, or `channel_type=private_channel`). Private messages were shared in a restricted context and must not be surfaced into a public or different channel reply.
- If a result's channel cannot be determined, discard it rather than risk exposure.
- Only results from **public channels** may be used as sources in a wide-search reply.
**Cross-channel attribution — MANDATORY:**
- When a source thread comes from a **different channel** than the question channel, always include the source channel as a clickable link in the bullet, e.g.: `<@RECOMMENDER> in <#SOURCE_CHANNEL_ID> (<permalink|source>)`.
- In the reply header, note that results came from across the workspace, e.g.: *"Similar questions came up in other channels — here's what people recommended:"*
- Never silently use a cross-channel source without attributing which channel it came from.
## Step 4 — Extract concrete recommendations / answers

From each prior thread, pull out:
- The specific name (shop, product, person, command, fact)
- One-line context (why it was recommended)
- Attribution: `<@USER_ID>` of the recommender
- Source permalink — to the specific message if possible, else the parent thread
Discard:
- Vague comments ("you'll be fine")
- Pure jokes or banter
- Stale info (e.g. "they closed in 2019" said about a shop)
If you end up with **zero concrete items**, stop and tell the user: "No useful prior answers in this channel — nothing to draft."

## Step 4b — Resolve Slack user IDs

Before composing the reply, resolve every person you intend to mention (question author + all recommenders) to their Slack user ID using `slack_search_users`. Use their full display name as the search query.

- If a user ID was already returned in the thread read (e.g. the message metadata includes `user_id`), use that directly — no lookup needed.
- If the lookup returns a confident single match, use `<@USER_ID>` in the reply.
- If the lookup returns multiple ambiguous matches or no match, fall back to the plain display name (no `@` mention) rather than tagging the wrong person.
- Never guess or construct a user ID. Never mention someone as `<@firstname.lastname>` — only use IDs returned by `slack_search_users`.
## Step 5 — Compose the reply

### For `RECOMMENDATION_GATHERING`

Slack-formatted, ≤900 chars. Structure:

```
Hi <@QUESTION_AUTHOR>! Same question came up before — here's what people in <#CHANNEL> recommended:

• *<Name 1>* — <one-line context>. <@RECOMMENDER_USER_ID> (<source permalink|source>)
• *<Name 2>* — <one-line context>. <@RECOMMENDER_USER_ID> (<source permalink|source>)
• *<Name 3>* — <one-line context>. <@RECOMMENDER_USER_ID> (<source permalink|source>)

Full prior thread: <PARENT_THREAD_PERMALINK|here>.
```

Do NOT add a `Sent using` footer — it is appended automatically by the posting automation.

### For `SIMPLE_QUESTION`

Only post if you found a **confident, correct** answer. "Confident" means at least one of:
- ≥2 independent people in past threads gave the same answer
- One authoritative reply (e.g. from the team owner / a clearly-knowledgeable person, no contradictions)
Format (1–3 sentences):

```
Hi <@QUESTION_AUTHOR>! <Direct answer in plain English>.
Source: <PERMALINK|prior thread>.
```

If confidence is below the bar, **do not draft** — tell the user "found possible answers but not confident enough to post" and list what you found.

## Step 6 — Show the draft and ask the user what to do

Always print the composed reply to the user first. **Never post without an explicit go-ahead in the same turn.** Show:
- Channel + thread you're replying to (with permalinks)
- Classification
- Number of source threads used
- The reply body verbatim, in a fenced code block so the user can copy it
Then ask exactly one question, e.g.:

> "Post this as a thread reply to <thread permalink|the question>, or just keep the text here?"

Wait for the user's answer:
- If they say "post" / "send" / "yes, post" / equivalent → before calling `slack_send_message`, re-read the thread with `slack_read_thread` to check whether a reply containing `Sent using` and `U0AE83PNC8M` already exists. If it does, **do not post** — tell the user "A Claude-authored reply already exists in this thread — skipping to avoid duplication" and show the existing reply's permalink. Only proceed with `slack_send_message` if no such reply exists. Use `channel_id=<CHANNEL_ID>`, `thread_ts=<question_ts>`, `reply_broadcast=false` (true only if they explicitly asked to broadcast). After posting, return the permalink to the new reply.
- If they say "just text" / "no" / "keep it" / "draft only" / equivalent → do nothing further. The draft already shown is the deliverable.
- If they ask for edits → revise and re-show the draft, then ask again.
Do not interpret silence, ambiguity, or partial confirmation ("looks good") as permission to post. Re-ask if unclear.

## Constraints

- **Never post automatically.** Always prepare the draft, show it, and ask the user whether to post it or just keep the text. Posting only happens after explicit confirmation.
- **Never invent recommendations.** Every bullet must trace to a real prior message — include the permalink so the user can verify.
- **Never use this skill on for-sale messages, announcements, or off-topic chat.** Stop and explain.
- **Hard caps per invocation:** 3 search calls, 5 thread reads, 1 outbound reply (only if the user confirms posting).
- **Never include a `Sent using` footer.** It is appended automatically by the posting automation — do not add it manually. Before finalising the message string, scan it and remove any occurrence of `Sent using` to avoid duplication.
- **Never post a duplicate Claude reply.** Before posting, re-read the thread and check for any existing reply containing both `Sent using` and `U0AE83PNC8M`. If found, abort and inform the user instead of posting.
- **Always tag people with their real Slack user ID.** Use `slack_search_users` to resolve every name to a `<@USER_ID>` before composing the reply. Never use plain text names where a resolved ID is available. Fall back to plain name only if lookup is ambiguous or returns no match — never guess an ID. Results from private channels must be silently discarded — do not quote, summarise, or permalink them. Only public channel results may appear in replies.
- **Wide search: always attribute the source channel.** Any bullet sourced from a different channel than the question channel must include a `<#CHANNEL_ID>` link. No silent cross-channel sourcing.
- **Use Slack formatting:** `<URL|label>` for links, `<@USERID>` for mentions, `*bold*`, `_italic_`. Don't use Markdown `[label](url)` — Slack won't render it.
- **Respect the user's posting-rule preference:** if they have a "search before posting" rule, the search is built into Step 3 — but still surface the draft for confirmation.
## Edge cases

- **Channel only contains the current question.** Tell the user there's no prior history — nothing to summarise.
- **The question itself is a follow-up inside an existing thread.** Treat the *thread parent* as the question, not the follow-up reply, and search for threads similar to the parent. If the user wants to answer the follow-up specifically, ask for clarification.
- **The author retracted or edited the question.** Use the latest text. If now empty, stop.
- **Multiple plausible answers contradict each other.** For `SIMPLE_QUESTION`, this fails the confidence bar — fall back to listing the options as if it were `RECOMMENDATION_GATHERING`.
- **Channel is private and not in the user's membership.** `slack_search_public_and_private` will return nothing. Inform the user.
- **Wide search returns only private-channel results.** If all cross-channel results are from private channels (and thus discarded), tell the user: "Found matches but all were in private channels — cannot surface them." Do not draft a reply.
- **Claude reply already exists in the thread.** If a prior reply containing `Sent using` + `U0AE83PNC8M` is found when re-reading the thread before posting, abort with: "A Claude-authored reply already exists in this thread — skipping to avoid duplication." Show the existing reply's permalink.
## Example invocation

User: "Find answer for this slack thread: https://taxify.slack.com/archives/C09RRHCKR3J/p1778056897398289"

Skill flow:
1. Parse: channel=`C09RRHCKR3J`, ts=`1778056897.398289`.
2. Read thread → "place for pre-purchase car inspection, Toyota, Elke is full".
3. Classify → `RECOMMENDATION_GATHERING`.
4. Search `in:<#C09RRHCKR3J>` for "pre-purchase inspection", "auto shop check before buying", "used car inspection". Find Abhinav's thread (Feb 28) with two named recommendations.
5. Extract: bilservice.ee (Alexander Blinov), Valerii / autolimb.ee (Maks Petrenko).
6. Compose bulleted reply with attributions and permalinks.
7. Show draft in a fenced code block + summary (channel, thread, classification, sources).
8. Ask: "Post this as a thread reply, or just keep the text here?"
9. Wait for user reply. If "post" → send with `slack_send_message`. If "just text" or anything else → stop.
## Notes for callers

- This skill is independent and channel-agnostic. For a recurring auto-answer bot scoped to a single channel, pair this skill with a scheduled task that supplies the channel ID and the new-message detection loop.
- The skill does not maintain state between invocations — it is one-shot per question. State (idempotency, dedup) is the caller's responsibility.
