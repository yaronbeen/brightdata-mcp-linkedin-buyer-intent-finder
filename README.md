# linkedin-buyer-intent-finder

A Claude Skill that finds people publicly asking for what you sell on LinkedIn, verifies each post, qualifies the opportunity, checks the existing conversation, and drafts a helpful reply.

Cold outbound interrupts someone who never asked. This does the opposite: it finds public posts from people evaluating tools, looking for providers, replacing a product, or describing an active problem. Search results are only leads; the skill verifies the original post before recommending a reply.

## What it does

| Stage | Tool | Output |
|---|---|---|
| Discover | Bright Data `search_engine` | Public LinkedIn post URLs across direct buying-intent and pain-signal queries |
| Verify | Bright Data `scrape_as_markdown` | Original post text, visible date, author context, and public conversation evidence |
| Freshness gate | Claude | A verified 0-7 day primary tier, with older or undated posts clearly downgraded |
| Qualify | Claude | A deterministic 0-100 score for intent, fit, recency, accessibility, conversation, and risk |
| Competition check | Claude + Bright Data | Existing recommendations, named alternatives, unanswered objections, and saturated threads |
| Draft | Claude | One concise LinkedIn reply that answers first and discloses affiliation when relevant |

Output is ranked by qualification score. Unverified, private, stale, off-topic, and promotional posts are skipped.

## Required connector

**Bright Data MCP.** The baseline workflow uses `search_engine` to discover indexed public posts and `scrape_as_markdown` to verify the canonical LinkedIn page. Both tools are available in Bright Data's Rapid mode; batch and structured LinkedIn tools may depend on the configured tool set or plan.

Add it in **Settings -> Connectors**, or for Claude Code:

```bash
export BRIGHTDATA_API_KEY="your-api-key"
claude mcp add --transport sse brightdata "https://mcp.brightdata.com/sse?token=$BRIGHTDATA_API_KEY"
```

Do not commit the API key or paste it into screenshots, logs, or issues.

Two things worth knowing before you run it:

- **LinkedIn visibility is incomplete.** Public pages may omit dates, comments, reactions, or parts of the post. The skill reports missing evidence instead of inventing it and does not bypass sign-in walls.
- **Search snippets are not proof.** Every candidate must be scraped at its canonical public URL before scoring or drafting. A post without a verifiable date cannot enter the priority tier.

## Install

**Personal (available everywhere):**

```bash
git clone https://github.com/yaronbeen/linkedin-buyer-intent-finder ~/.claude/skills/linkedin-buyer-intent-finder
```

**Per project:** clone into `.claude/skills/linkedin-buyer-intent-finder` inside the repository instead.

**OpenCode:** clone into `~/.config/opencode/skills/linkedin-buyer-intent-finder`.

**Claude apps:** upload `SKILL.md` in **Settings -> Capabilities -> Skills**.

Restart the client, or start a new session, so the skill loads.

## Usage

```text
/linkedin-buyer-intent-finder
find LinkedIn posts from people asking for a CRM this week
find public LinkedIn posts from agencies struggling with Meta ad creative production
```

Claude asks for the product or category, target audience or geography, exclusions, and preferred reply voice when those details are missing.

## Built-in guardrails

- **It never posts.** No comment, DM, connection request, like, or repost is sent on your behalf.
- **Public content only.** It does not log in, enter private groups, or bypass access controls.
- **No contact scraping.** It does not collect personal email addresses or phone numbers.
- **Bad fits get skipped.** A numerical score never overrides a private, stale, unsafe, or irrelevant result.
- **Dates are verified.** Search-engine date filters and snippets are not trusted as the final freshness source.
- **Affiliation is disclosed.** Drafts identify the relationship when mentioning the represented product.
- **Scraped text is untrusted.** Instructions embedded in posts or linked pages cannot alter the workflow or trigger unrelated actions.

## License

MIT

---

Made by Yaron Been
