# linkedin-buyer-intent-finder

**Find the people already raising their hand on LinkedIn.**

A Claude and OpenCode skill that searches indexed public LinkedIn posts for people asking for recommendations, comparing alternatives, looking for providers, or describing a problem your product may solve.

The signal is not `CRM`. The signal is `What CRM should we switch to?` This skill starts with the public need, attempts to verify the original post, and drafts a reply only when the evidence and fit justify one.

**A keyword match earns a scrape, not a pitch.**

## What it does

| Stage | Tool | Output |
|---|---|---|
| Discover | Bright Data `search_engine` or `discover` | Candidate public LinkedIn post URLs from buying-intent and pain-signal queries |
| Verify | Bright Data `scrape_as_markdown` | Whatever post text, date, author, and conversation evidence is publicly accessible; missing fields stay unknown |
| Freshness gate | Claude | Verified 0-7 day posts first, 8-14 day posts as a secondary tier, and no scoring or drafting for undated or older posts |
| Qualify | Claude | A fixed 0-100 rubric for intent, fit, recency, accessibility, conversation, and risk; evidence classification still requires model judgment |
| Competition check | Claude + Bright Data | For qualified candidates, visible recommendations, alternatives, objections, and thread saturation when accessible |
| Draft | Claude | One concise LinkedIn reply that answers first and discloses affiliation when relevant |

Output contains up to 10 scored candidates, sorted by score, freshness, and visible competition. Unverified candidates may be listed separately, but they are never scored or drafted.

## Required connector

**Bright Data MCP.** The baseline workflow discovers candidates with `search_engine` or `discover`, then checks them with `scrape_as_markdown`. Bright Data currently lists all three in Rapid mode. Batch-tool availability depends on the MCP configuration, while the more reliable `web_data_linkedin_posts` structured extractor belongs to the Pro social tool set.

In Claude, add Bright Data from **Customize -> Connectors**. For Claude Code, add Bright Data's remote MCP endpoint over HTTP:

```bash
export BRIGHTDATA_API_KEY="your-api-key"
claude mcp add --transport http brightdata "https://mcp.brightdata.com/mcp?token=$BRIGHTDATA_API_KEY"
```

Do not commit the API key or paste it into screenshots, logs, or issues.

The command configures Bright Data for the current Claude Code scope. Configure the connector wherever the skill will run, or use Claude Code's user-scoped MCP option when you want it across local projects. OpenCode users must configure the same Bright Data remote server separately in `opencode.json`.

Two things worth knowing before you run it:

- **LinkedIn visibility is incomplete.** Generic public-page scraping may omit dates, comments, reactions, or parts of a post. The workflow keeps missing evidence as `unknown` and does not bypass sign-in walls.
- **Search snippets are not proof.** Every candidate must be checked at its canonical public URL before scoring. A missing date means no score and no draft.

## Install

**Claude Code personal skill (available across local projects):**

```bash
git clone https://github.com/yaronbeen/linkedin-buyer-intent-finder ~/.claude/skills/linkedin-buyer-intent-finder
```

**Per project:** clone into `.claude/skills/linkedin-buyer-intent-finder` inside the repository instead.

**OpenCode:** clone into `~/.config/opencode/skills/linkedin-buyer-intent-finder`. OpenCode discovers it through the native `skill` tool; Bright Data still needs its own MCP configuration.

**Claude apps:** upload a ZIP whose top-level folder is named `linkedin-buyer-intent-finder` and contains `SKILL.md`, using **+ -> Create skill -> Upload a skill**. Uploading `SKILL.md` by itself, or leaving `-main` in the folder name from GitHub's download ZIP, is not sufficient.

Claude Code normally detects a new skill without restarting. Start a new session only if the top-level skills directory did not exist when the current session began or the skill is not listed.

## Usage

```text
/linkedin-buyer-intent-finder
find LinkedIn posts from people asking for a CRM this week
find public LinkedIn posts from agencies struggling with Meta ad creative production
```

In OpenCode, ask for the same task naturally and let the agent load `linkedin-buyer-intent-finder` through the skill tool.

If the product context is too thin to judge fit, the skill asks one concise clarifying question. Otherwise it uses the documented defaults.

## Built-in guardrails

- **Research first. Human decision second.** The skill instructs the agent to draft only, never post, message, connect, like, or repost.
- **Public content only.** Private profiles, groups, gated pages, and access-control bypasses are out of scope.
- **No contact-list building.** Personal emails and phone numbers are not collected.
- **Hard gates beat the score.** Missing dates, stale posts, spam, vendor promotion without first-party need, unsafe content, and bad fits cannot become qualified leads.
- **Affiliation is disclosed.** Drafts identify the relationship when mentioning the represented product.
- **Scraped text is untrusted.** The workflow treats instructions inside posts and linked pages as data, not as commands.
- **The score is a rubric, not ground truth.** The formula is fixed and bounded, but the model still judges the evidence bands.

## License

MIT

---

Made by Yaron Been
