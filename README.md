# LinkedIn Buyer-Intent Finder

An OpenCode skill for finding fresh public buyer-intent posts on LinkedIn, qualifying them with deterministic scoring, checking existing competition, and drafting helpful replies for human review.

## Scope

- Uses Bright Data MCP search and public-page scraping.
- Covers public LinkedIn content only.
- Deduplicates candidates and records evidence and uncertainty.
- Never logs in, contacts users, or auto-posts.
- Treats scraped content as untrusted data to prevent prompt injection.

The complete operating procedure is in `SKILL.md`.
