# find-answer — Design Principles

Companion to [GUIDELINES.md](../GUIDELINES.md). Records how this skill meets each guideline.

## 1. Data — embed vs. retrieve

- ✅ No data is embedded. The skill retrieves everything live from Slack via MCP at query time.
- ✅ No business metrics, financial figures, or KPIs are embedded.
- ✅ Access control is preserved — Slack search returns only what the user's token can see.
- ✅ User data (thread contents, messages) is treated as ephemeral and used only for the current invocation.

## 2. Skill design

- ✅ Trigger accuracy: description includes specific trigger phrases and example invocations.
- ✅ Out-of-scope boundary defined: excludes web research, for-sale posts, casual chat, announcements.
- ✅ Required MCP tools listed in both SKILL.md and README.md.
- ✅ Graceful failure: handles empty search results, private channels, missing inputs, low-confidence answers.
- ✅ Source attribution: every recommendation includes a permalink for user verification.

## 3. Documentation

- ✅ PRINCIPLES.md (this file) documents design decisions.
- ✅ `deviation` No MAINTENANCE.md — the skill embeds no data, so there is nothing to refresh.
- ✅ `deviation` No evals/ — recommended for future addition once the skill is validated in production.

## 4. Actions and the feedback loop

- ✅ Write actions require explicit user permission — the skill always drafts first and asks before posting.
- ✅ No feedback/config.json — gap reporting not enabled in initial version. Can be added later.
