# PostWire

Social publishing and scheduling API with a hosted MCP server. From one idea it writes a separate draft for each network (length, hashtags, title and tags, link placement), then publishes or schedules them through the accounts the user connected.

> **Disclosure:** this guide was contributed by the maker of PostWire. It is not a sponsored or partner listing. PostWire is a young product (2026); check the current network list, limits and prices at https://postwire.io/pricing.md before relying on it.

## Capabilities

| Integration | Available | Notes |
|-------------|-----------|-------|
| API | ✓ | REST API; OpenAPI at https://postwire.io/openapi.json |
| MCP | ✓ | Hosted Streamable HTTP server at `https://postwire.io/api/mcp`, OAuth 2.1 sign-in or API key |
| CLI | - | Not available (an n8n community node, `n8n-nodes-postwire`, exists) |
| SDK | - | Use REST or MCP |

Networks: TikTok, Instagram (business or creator accounts), Facebook Pages, YouTube, LinkedIn (personal profiles, not company Pages), Bluesky, Mastodon, Telegram, Discord; X on paid plans (pay per post with credits). Not available: Threads, Reddit, Pinterest.

## Authentication

- **MCP**: OAuth 2.1 with dynamic client registration; the user signs in in the browser and no key is copied
- **API**: `Authorization: Bearer {api_key}`, key created in the dashboard (API & MCP) and kept in `POSTWIRE_API_KEY`
- **Get access**: https://postwire.io (free plan, no card)

## Setup

### MCP

```bash
# Claude Code
claude mcp add --transport http --scope user postwire https://postwire.io/api/mcp
# Codex
codex mcp add postwire --url https://postwire.io/api/mcp
```

Other clients (Cursor, VS Code, Windsurf, Gemini CLI): https://postwire.io/mcp/

Main tools: `my_account`, `generate_posts` (drafts only), `post_to_social`, `schedule_post`, `bulk_schedule` (dry run by default), `list_scheduled_posts`, `cancel_scheduled_post`, `create_connect_link`, `get_post_performance`.

## Common Agent Operations

### Check plan and connected networks

```bash
GET https://postwire.io/api/me
Authorization: Bearer {api_key}
```

### Write one draft per network (nothing is published)

```bash
POST https://postwire.io/api/generate
Content-Type: application/json

{"prompt": "We now open at 7 on Saturdays", "platforms": ["linkedin", "bluesky", "instagram"]}
```

Returns `{ "drafts": { "<network>": { "text", "title?", "tags?" } } }`. Show them to the user before publishing.

### Publish the approved drafts

```bash
POST https://postwire.io/api/post
Content-Type: application/json

{"platforms": ["linkedin", "bluesky"], "per_platform": {"linkedin": {"text": "..."}, "bluesky": {"text": "..."}}, "idempotency_key": "sat-hours-1"}
```

One result per network; one failing does not cancel the others.

### Schedule in the next free queue slot

```bash
POST https://postwire.io/api/schedule
Content-Type: application/json

{"platforms": ["linkedin", "bluesky"], "text": "...", "run_at": "next_slot", "timezone": "America/Lima"}
```

### Check, then schedule a content calendar

```bash
POST https://postwire.io/api/bulk
Content-Type: application/json

{"timezone": "America/Lima", "dry_run": true, "rows": [
  {"date": "2026-11-02 09:00", "networks": "linkedin, bluesky", "text_linkedin": "...", "text_bluesky": "...", "first_comment": "https://example.com/guide"}
]}
```

The dry run returns each row's time, each network's length as that network counts it, whether it waits for approval or is held by the plan, and errors. Send again with `"dry_run": false`: all rows are scheduled or none. `DELETE /api/bulk/{batch_id}` undoes the upload.

### Read what performed

```bash
GET https://postwire.io/api/insights?tz=America/Lima
```

Posts ranked against the same account's median. Numbers come from Bluesky, Mastodon, Instagram (likes and comments) and YouTube (views); TikTok, LinkedIn, Facebook, Telegram and Discord are listed as not measurable.

## Approvals

A brand can require a person to approve posts from AI agents, API keys or automations, or only posts with a link or price. Such a post answers `202` with `status: "pending_approval"`; API keys and MCP clients cannot approve it.

## Pricing

| Plan | Cost | Includes |
|------|------|----------|
| Free | $0 | 1 brand, 20 posts/month, 2 networks per post, 3 scheduled posts at a time |
| Starter | $9/mo | 3 brands, 1,000 posts/month, every network in each post |
| Pro | $29/mo | 10 brands, unlimited posts (fair use), 25 seats |
| Agency | $99/mo | 50 brands |
| Scale | $299/mo | 200 brands |

Priced per brand, not per network. Current prices: https://postwire.io/pricing.md

## Rate Limits

- API requests per key per minute: Free 60, Starter 120, Pro 300, Agency 600, Scale 1,200
- `429` with `retry_after`; every error has `code`, `what` and `fix`: https://postwire.io/docs/errors.md

## When to Use

- The agent should write a different version of a post for each network rather than send one text everywhere
- Publishing or scheduling from an MCP client (Claude, Codex, Cursor) with browser sign-in instead of keys
- Queueing a content calendar with a dry run before anything is stored
- TikTok and YouTube video uploads from an agent or automation

## When to Use Something Else

- **Buffer**: the team already schedules there, or needs networks PostWire does not post to (LinkedIn company Pages, Threads, Pinterest); see [buffer.md](buffer.md)
- **Manual posting or the networks' own schedulers**: one or two accounts and a few posts a week
- **The network's own API**: deep control of a single network (ads, DMs, analytics beyond post metrics)

## Links

- Docs: https://postwire.io/docs/
- MCP: https://postwire.io/mcp/
- Agent-readable docs: https://postwire.io/llms.txt

## Relevant Skills

- social
- content-strategy
- launch
- video
