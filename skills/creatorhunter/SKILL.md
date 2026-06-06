---
name: creatorhunter
description: Find, match, hunt, analyze, and reach out to social-media creators (TikTok / YouTube / Instagram) for brand and influencer campaigns through the CreatorHunter MCP server. Use whenever the user wants to discover creators for a brand, build a shortlist from a campaign brief, hunt down niche/under-the-radar talent, analyze their saved matches, or draft and send creator outreach.
metadata:
  homepage: https://app.creatorhunter.io/ai-agent
  category: marketing
---

# CreatorHunter

CreatorHunter is a creator-discovery and influencer-outreach platform. This skill
connects an AI agent to a user's CreatorHunter account through CreatorHunter's
remote MCP server, so the agent can search and match creators, hunt down niche
talent on demand, reason over the user's saved matches, and draft + send
personalized outreach — all backed by the same matching engine that powers the
CreatorHunter app.

API access is an **Apex plan** feature. If tools return "API access requires the
Apex plan", tell the user to upgrade at https://app.creatorhunter.io.

## Setup — connect the MCP server (do this first)

The CreatorHunter MCP server is remote and authenticated with the user's
**personal API key**. The key is account-scoped and acts on the user's behalf, so
never log it, echo it back in plaintext, or commit it.

1. **Get a key.** Ask the user to open
   https://app.creatorhunter.io/ai-agent, create an API key, and paste it to you.
   (Keys are shown only once.) If they already added the server, skip to "Using
   the tools".

2. **Register the server.** Use the user's agent's MCP mechanism. For Claude
   Code, run:

   ```bash
   claude mcp add --scope user --transport http creatorhunter \
     https://app.creatorhunter.io/api/mcp \
     --header "Authorization: Bearer THE_USERS_API_KEY"
   ```

   For **Codex**, run (Codex has no inline-header flag — it reads the key from an
   env var, so set `CREATORHUNTER_API_KEY` in the user's shell, e.g. `~/.zshrc`):

   ```bash
   export CREATORHUNTER_API_KEY="THE_USERS_API_KEY"
   codex mcp add creatorhunter \
     --url https://app.creatorhunter.io/api/mcp \
     --bearer-token-env-var CREATORHUNTER_API_KEY
   ```

   For **Gemini CLI**, run:

   ```bash
   gemini mcp add --transport http creatorhunter \
     https://app.creatorhunter.io/api/mcp \
     --header "Authorization: Bearer THE_USERS_API_KEY"
   ```

   For Claude Desktop, Cursor, or any other MCP client, add this to the MCP
   servers config instead:

   ```json
   {
     "mcpServers": {
       "creatorhunter": {
         "type": "http",
         "url": "https://app.creatorhunter.io/api/mcp",
         "headers": { "Authorization": "Bearer THE_USERS_API_KEY" }
       }
     }
   }
   ```

3. **Confirm.** Once connected, call `get_entitlements` to confirm the key is
   valid and see which capabilities (directory, outreach, match caps) the plan
   allows before relying on them.

## Using the tools

The server exposes these tools (all prefixed `creatorhunter` by the client). Tool
results are JSON. To protect the dataset, responses never reveal the total DB
size — only the returned page and its length — so don't infer totals or claim a
creator "doesn't exist" from an empty page.

### Discovery & matching
- **`match_creators`** — instant ranking of creators **already in CreatorHunter**
  against a brand/campaign. Returns each creator's match score, the signals they
  matched on, and standout recent videos. Best first choice for fast results.
- **`hunt_creators`** — deep hunt that *also* surfaces hard-to-find,
  under-the-radar talent in niche corners a quick search misses. May run in the
  background and return a `huntId` with status `hunting`; in that case poll
  **`hunt_status`** with that `huntId` and the **same** filters every ~30–60s
  until status is `complete`. Deep hunts are capped per day
  (`dailyHuntRemaining`); polling `hunt_status` is free.
- **`search_creators`** — direct directory browse/filter (no relevance ranking):
  free-text `search`, `categories`, `platforms`, `tiers`, follower range, min
  engagement, sorting, and cursor pagination (pass back `nextCursor`). Use for
  precise filtered lookups.
- **`get_creator`** — full profile for one creator by `creatorId` (bio,
  categories, every platform with follower counts, engagement / median views /
  hidden-gem score / growth, and recent videos with stats, hashtags, transcript
  snippets). Use to deep-dive after a match/hunt/search.

**Category is the dominant matching signal.** Always pass at least one
`categories` value for `match_creators` / `hunt_creators` / `search_creators`, and
use the exact names from **`list_categories`**. Use **`list_tiers`** for valid
`tiers` values (nano / micro / midTier / macro / mega). `keywords` and
`description` refine the ranking.

### The user's own data
- **`my_matches`** — the creators in the user's **own** matched list(s), with
  followers, engagement rate, median views, hidden-gem score, match score, and
  which list each belongs to. Use whenever the user says "my matches", "my list",
  or asks who to prioritize. `sort` accepts `match` (default) / `engagement` /
  `followers` / `views` / `score`.
- **`get_lists`** — the user's saved lists (name, size, id).
- **`save_to_list`** — save creators (e.g. the best from a match/hunt) into a
  list; pass `listName` to create-or-append, or `listId` to target an existing
  one. De-duplicates automatically.
- **`get_brands`** — the user's configured brand(s): product, goal, partnership
  types/budget, target niches/platforms, and a ready-made summary. **Call this
  before matching or drafting outreach** so you personalize from the user's real
  product details instead of asking them to retype everything.

### Outreach (Apex + a connected inbox)
Outreach follows a strict **draft → review → send** flow. Only the user's own
**unlocked** matches are eligible.

1. **`draft_outreach`** — write personalized emails for `creatorIds` given a
   `brief` (offer/angle) and optional `tone`. Returns a `campaignId`. Pull the
   brand context from `get_brands` first so the brief reflects the real offer.
2. **`get_outreach`** — review the drafts (recipient, subject, body, status)
   using the `campaignId`. Always show these to the user before sending. With no
   `campaignId`, lists recent campaigns.
3. **`send_outreach`** — by default (`live` omitted/false) saves the drafts to
   the user's email **Drafts** folder for a final human check — nothing is sent.
   Set `live: true` to actually email creators (respects a daily cap). **Never
   send live without explicit user confirmation.**
- **`get_contacted`** — creators already drafted/sent to; check this to avoid
  double-contacting.

### Plan & metadata
- **`get_entitlements`** — the user's plan and what the key can do.
- **`list_categories`** / **`list_tiers`** — canonical category and tier names.

## Recommended workflows

**Shortlist from a brief**
1. `get_brands` (and `list_categories`) → derive categories + keywords.
2. `match_creators` for instant results; if the user wants more/niche reach or
   the matches are thin, `hunt_creators` (poll `hunt_status` until complete).
3. Present a ranked shortlist with each creator's match score and *why* they
   matched. Offer to `save_to_list`.

**Analyze my matches**
1. `my_matches` with the relevant `sort` (e.g. `engagement`).
2. `get_creator` on the top few to reason over content themes, hidden-gem scores,
   and viral efficiency; tell the user who to prioritize and why.

**Outreach a shortlist**
1. `get_brands` for the offer; confirm the brief with the user.
2. `draft_outreach` on the chosen unlocked creators → `get_outreach` to review →
   show the user → `send_outreach` (default = Drafts; `live: true` only after
   explicit confirmation). Check `get_contacted` first to avoid duplicates.

## Notes
- Errors like "Apex plan required", "No connected inbox", "Invalid or revoked API
  key", or "creator must be unlocked" are actionable — relay them plainly and
  tell the user how to resolve them (upgrade, connect an inbox at
  https://app.creatorhunter.io, mint a new key, or unlock the creator).
- Prefer `match_creators` for speed and `hunt_creators` when reach/niche coverage
  matters; they share the same scoring, so results are comparable.
