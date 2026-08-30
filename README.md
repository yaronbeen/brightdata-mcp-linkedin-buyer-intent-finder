# brightdata-mcp-linkedin-buyer-intent-finder

**What if your next buyer has already posted the brief on LinkedIn?**

Not `CRM`.

`What CRM should we switch to before renewal?`

One is a keyword. The other may be intent.

This Claude and OpenCode skill searches indexed public LinkedIn posts for people asking for recommendations, comparing alternatives, looking for providers, or describing a problem your product may solve.

**It requires Bright Data MCP.** Rapid mode supports the baseline workflow; structured LinkedIn extraction is optional.

Then it does the part a keyword alert skips: it attempts to verify the original post, checks freshness and fit, looks at the visible conversation, and drafts a useful reply only when the evidence earns one.

**A match earns a scrape, not a pitch.**

## What exactly are we hunting?

Public posts where someone has already put a relevant need into words.

The skill looks for buying language and live pain signals, then filters out stale posts, hiring and recruiting posts, sellers prospecting for their own clients, vendor and competitor promotions, generic thought leadership, bad fits, and threads where another pitch would add nothing.

It does not promise to find every relevant LinkedIn post. Search indexes miss things. LinkedIn hides things. Public pages sometimes arrive with half the useful context missing.

That limitation is not fine print. It is part of the workflow.

## How does a noisy post earn a place on the list?

| Stage | Tool | Output |
|---|---|---|
| Discover | Bright Data `search_engine` or `discover` | Candidate indexed public LinkedIn post URLs from buying-intent and pain-signal queries |
| Verify | Bright Data `scrape_as_markdown` | Whatever post text, date, author, and conversation evidence is publicly accessible; missing fields stay `unknown` |
| Freshness gate | Claude | Verified 0-7 day posts first, 8-14 day posts as a secondary tier, and no scoring or drafting for undated or older posts |
| Qualify | Claude | A fixed, bounded 0-100 rubric for intent, fit, recency, accessibility, conversation, and risk; classification still requires model judgment |
| Competition check | Claude + Bright Data | For qualified candidates, visible recommendations, alternatives, objections, and thread saturation when accessible |
| Draft | Claude | One concise LinkedIn reply that answers first and discloses affiliation when relevant |

The report contains up to 10 scored candidates by default. It sorts them by score, freshness, and lower visible competition.

Unverified candidates may appear separately. They get no score and no draft.

### What does one result look like?

Illustrative format only, not a real lead or performance claim:

```text
QUALIFIED - 74/100 - LinkedIn
Posted: 2 days ago (verified)
Intent: Asking which CRM to replace before renewal
Fit: Direct category match
Competition: Two visible recommendations; one objection unanswered
Draft: A concise answer with one practical tradeoff and disclosed affiliation
```

## How fresh is fresh enough?

- `0-3 days`: HOT when intent and fit are also strong.
- `4-7 days`: ACTIVE and part of the primary tier.
- `8-14 days`: AGING and secondary. It can rank no higher than QUALIFIED.
- `15-30 days`: ARCHIVE. No score. No draft.
- `>30 days`: SKIP.
- Unverifiable date: `freshness_unverified`. No score. No draft.

Relative dates are used only at the precision the visible label supports. If a range crosses a 3-, 7-, or 14-day boundary, the older tier wins. A coarse label never becomes a made-up timestamp. Only the main post's own date counts: dates from LinkedIn's "More Relevant Posts" feed are ignored, and if the main date cannot be isolated the post is treated as undated.

## Can a score override a bad fit?

No.

The arithmetic is fixed and clamped to `0-100`:

```text
score = max(0, min(100, intent + fit + recency + accessibility + conversation - risk))
intent:         0-35
fit:            0-25
recency:        0-20
accessibility: 0-10
conversation:   0-10
risk:           0-20 (subtracted)
```

The rubric weighs intent, fit, recency, accessibility, visible conversation, and risk. The model still has to classify the evidence, so the score is a consistent review framework, not objective truth.

Fit is a floor, not just a weight: a weak-adjacency post (fit 5 or below) can never rank above WATCH and never gets a draft, no matter how strong the intent. That is what stops a high-intent post that wants a different thing than you sell from becoming a lead.

Other hard exclusions beat the number every time: private or gated content, sensitive personal data, malicious requests, spam, vendor promotion without first-party buying need, a hiring or reverse-seller post outside your offer, an unverifiable date, age over 14 days, a direct mismatch, a response requiring high-stakes advice, or a saturated thread with nothing new to add.

## What does "verified" honestly mean?

Best-effort verification of the canonical public LinkedIn URL.

When accessible, the workflow records the post text, date and evidence source, relevant public author context, visible replies, competitor mentions, engagement, discovery query, and scrape timestamp.

LinkedIn may omit some or all of that. Missing facts stay `unknown`. The workflow does not bypass sign-in walls or infer what it cannot see.

Search snippets are clues. They are not proof.

## Which Bright Data setup do you need?

**Bright Data MCP is required.**

The baseline path uses:

1. `search_engine` for discovery and follow-up searches.
2. `discover` as an alternative discovery tool.
3. `scrape_as_markdown` to check canonical public posts.

Bright Data currently lists those tools in Rapid mode. Batch-tool availability depends on the MCP configuration.

The optional `web_data_linkedin_posts` structured extractor is available through Bright Data's social tool set, which can be enabled directly or through full Pro mode. Bright Data describes structured extractors as often faster and more reliable than generic scraping, but they are not required for the baseline path.

See the current [Bright Data MCP tools reference](https://docs.brightdata.com/ai/mcp-server/tools).

## How do you connect Bright Data?

In supported Claude plans and surfaces, add a custom Bright Data connector from **Customize -> Connectors**. Team or Enterprise owners may need to add or enable the connector for the organization.

For Claude Code, add the remote MCP endpoint over HTTP:

```bash
export BRIGHTDATA_API_KEY="your-api-key"
claude mcp add --transport http brightdata "https://mcp.brightdata.com/mcp?token=$BRIGHTDATA_API_KEY"
```

That command uses the current Claude Code scope. Configure the connector wherever the skill runs, or use Claude Code's user-scoped MCP option across local projects.

OpenCode users can add the same server to `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "brightdata": {
      "type": "remote",
      "url": "https://mcp.brightdata.com/mcp?token={env:BRIGHTDATA_API_KEY}",
      "enabled": true
    }
  }
}
```

Installing the skill does not install its connector.

Do not commit the API key or paste it into logs, screenshots, or issues.

## How do you install it?

**Claude Code, across local projects:**

```bash
git clone https://github.com/yaronbeen/brightdata-mcp-linkedin-buyer-intent-finder ~/.claude/skills/brightdata-mcp-linkedin-buyer-intent-finder
```

**Claude Code, one project:** clone into `.claude/skills/brightdata-mcp-linkedin-buyer-intent-finder` inside that project.

**OpenCode:** clone into `~/.config/opencode/skills/brightdata-mcp-linkedin-buyer-intent-finder`. OpenCode discovers it through the native `skill` tool.

**Claude apps:** upload a ZIP whose top-level folder is named `brightdata-mcp-linkedin-buyer-intent-finder` and contains `SKILL.md`, using **+ -> Create skill -> Upload a skill**. A loose `SKILL.md` or a folder ending in `-main` will not package correctly. Code execution and file creation must be enabled; organization settings may require an owner to enable Skills.

Claude Code normally detects new skills live. Start a new session only if the top-level skills directory did not exist when the session began or the skill is not listed.

## What do you ask it to do?

```text
/brightdata-mcp-linkedin-buyer-intent-finder
find LinkedIn posts from people asking for a CRM this week
find public LinkedIn posts from agencies struggling with Meta ad creative production
```

In OpenCode, ask naturally and let the agent load `brightdata-mcp-linkedin-buyer-intent-finder` through the skill tool.

The product or category, intended buyer, and your relationship to the represented product are required before the product can appear in a draft. The skill gathers missing context in one concise question. Otherwise it defaults to any geography, excludes hiring and vendor-promotion noise, and uses a concise helpful voice.

## How does this avoid becoming spam?

A qualified post still has to survive the conversation check.

The draft must:

- answer the actual question before mentioning a product;
- add one concrete tactic, diagnostic question, example, or tradeoff;
- use only user-provided facts or verified sources for claims;
- disclose affiliation when mentioning the represented product;
- avoid fake experience, invented results, false scarcity, unverifiable pricing, and `DM me` bait;
- skip saturated threads when it cannot add something distinct.

No robotic `Great post!` opener. No product shoehorned into a problem it does not solve.

Research first. Human decision second.

## What will it never do?

The skill instructs the agent not to:

- log in or bypass access controls;
- post, comment, message, connect, like, or repost;
- scrape private profiles, private groups, or authenticated-only content;
- collect personal emails or phone numbers;
- infer sensitive traits or build hidden dossiers;
- turn an undated search snippet into a scored lead;
- claim that outreach or any platform action happened.

Scraped posts, profiles, replies, snippets, and linked pages are treated as untrusted data, not instructions. Suspected prompt injection is flagged and excluded from drafting.

Public availability is not permission. This skill does not decide whether automated access or reuse is allowed under LinkedIn's terms, a contract, or applicable law. Use it only where you are authorized to access and use the data.

## FAQ: what would a sensible skeptic ask?

### Does it find every relevant LinkedIn post?

No. It searches indexed public content, and both search coverage and LinkedIn's public visibility are incomplete.

### Is every discovered post verified?

No. Verification is best effort. Anything that cannot be checked stays in a separate unverified section with no score and no draft.

### Why reject a promising post with no date?

Because `fresh` cannot be proved. It could be from this morning or two years ago. The skill refuses to pretend those are equal.

### Is the score objective?

No. The formula, bounds, and hard gates are fixed. The model's evidence classification is still subjective.

### Is Bright Data Pro required?

No for the baseline. Rapid mode supports `search_engine`, `discover`, and `scrape_as_markdown`. The optional structured LinkedIn extractor is available through the social tool set or full Pro mode.

### Does it need a LinkedIn login?

No. It is limited to indexed public content and publicly accessible pages. Authenticated-only content is out of scope.

### Does it auto-post?

No. It researches and drafts for human review. Posting and outreach are intentionally outside the skill.

### What identifiable information appears in the report?

The report can include a public author name, relevant public role or company, and the post URL. It does not enrich or collect off-platform emails or phone numbers.

### What if the thread already has the same answer five times?

It looks for a distinct contribution. If there is none, it marks the thread as saturated instead of manufacturing another pitch.

### Can I ask for more than 10 candidates?

Yes, but more candidates require more verification work and MCP usage. Ten is the default review limit, and sparse evidence may produce fewer.

### How much does a run cost and how long does it take?

There is no fixed promise. It depends on the queries, candidate count, verification calls, Bright Data plan, rate limits, and host. Check Bright Data's current pricing and usage data for your setup.

### Does the skill store a prospect database?

No database or persistence layer ships in this repository. Your AI host and MCP provider may retain logs under their own policies, so review those settings before using sensitive business context.

### What if I want the same workflow for X?

Use [X Buyer Intent Finder](https://github.com/yaronbeen/brightdata-mcp-x-buyer-intent-finder). This repository supports LinkedIn only.

### Does this guarantee legal or platform compliance?

No. Public does not automatically mean permitted. Users must ensure their access and use comply with applicable terms, contracts, and law.

## What is the license?

MIT

---

Made by Yaron Been
