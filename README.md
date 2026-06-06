# CreatorHunter skill

[![Add to your agent](https://skills.sh/b/PauliusOS/creatorhunter-skill)](https://skills.sh/PauliusOS/creatorhunter-skill)

An [Agent Skill](https://skills.sh) that connects your AI agent to
[CreatorHunter](https://app.creatorhunter.io) — find, match, and hunt down
social-media creators (TikTok / YouTube / Instagram) for brand campaigns,
analyze your saved matches, and draft + send personalized outreach, all through
CreatorHunter's remote MCP server.

## Install

```bash
npx skills add PauliusOS/creatorhunter-skill
```

Then create a personal API key at
**https://app.creatorhunter.io/ai-agent** (Apex plan) and tell your agent — the
skill walks it through connecting the MCP server with your key.

## What your agent can do

- **Match** creators already in CreatorHunter against a brand or campaign brief
- **Hunt** down niche, under-the-radar talent on demand
- **Search** the creator directory with precise filters
- **Analyze** your own saved matches (engagement, hidden-gem score, …)
- **Outreach** — draft personalized emails, review them, then send (draft-safe by
  default)

See [`skills/creatorhunter/SKILL.md`](skills/creatorhunter/SKILL.md) for the full
tool reference and recommended workflows.

## Requirements

- A CreatorHunter account on the **Apex** plan (API access is Apex-only)
- An MCP-capable agent (Claude Code, Claude Desktop, Cursor, or any MCP client)
