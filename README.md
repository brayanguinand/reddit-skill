# Reddit Skill for Claude Code

A Claude Code skill that gives you real Reddit search from any session — no Reddit API credentials required.

Reddit blocks web crawlers and `site:reddit.com` doesn't work in search. This skill bypasses that by using two MCP servers that talk directly to Reddit's data: one for the archive (fast, no rate limits), one for real-time results. You get actual threads and comments, not SEO aggregators.

## Usage

Once installed, invoke it with:

```
/reddit <your question>
```

Examples:
```
/reddit best accounting software for freelancers in France
/reddit mechanical keyboard under $150 reddit
/reddit is Lisbon still affordable for expats in 2025
/reddit airfryer vs convection oven which is actually better
```

The skill targets the right subreddits automatically, fetches the most-discussed threads, reads the top comments, and returns a synthesized answer with source links — not a raw data dump.

## How it works

Two MCP servers with complementary coverage:

- **PullPush MCP** — searches the PullPush Reddit archive (coverage up to ~May 2025). No rate limits, fast. Used first and for the bulk of queries. Runs searches in parallel across subreddits.
- **Reddit Buddy MCP** — hits Reddit's live `.json` endpoints directly (no API key needed). Covers the last ~13 months that PullPush doesn't have. Used systematically as a complement, not just as a fallback.

The skill picks 1-3 relevant subreddits per query (never `r/all`), runs searches in parallel, reads comments on the 1-2 most relevant posts, and synthesizes. For "best of" queries it uses `browse_subreddit` with `sort=top` to get community-validated content directly.

### Why no Reddit API credentials?

Reddit's official API became paid in 2023 (the change that killed Apollo and most third-party clients). This skill doesn't use it:

- **PullPush** is an independent archive — it doesn't call Reddit at all at query time.
- **Reddit Buddy** reads Reddit's public `.json` endpoints (e.g. `reddit.com/r/python/top.json`), the same data your browser loads. These endpoints predate the API and Reddit can't easily close them without breaking their own site. Rate limit in anonymous mode: 10 req/min — sufficient for the ~3-4 calls a typical search makes.

## Prerequisites

Install both MCP servers globally:

```bash
npm install -g pullpush-mcp
npm install -g reddit-mcp-buddy
```

## Installation

**1. Add the MCPs at user scope** (makes them available in every Claude Code session):

```bash
claude mcp add pullpush /opt/homebrew/bin/pullpush-mcp --scope user
claude mcp add reddit-buddy /opt/homebrew/bin/reddit-mcp-buddy --scope user
```

Adjust the binary paths if npm installed them elsewhere (`which pullpush-mcp` to check).

**2. Copy `SKILL.md` to your Claude Desktop skills folder:**

On macOS:
```
~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/<plugin-id>/<session-id>/skills/reddit/SKILL.md
```

The easiest way is to use the Claude Code skill-creator skill, or manually drop the file into the `skills/` directory alongside your other skills (e.g. next to `ridgewood/` if you have it).

**3. Register the skill in `manifest.json`** in the same folder:

```json
{
  "skillId": "skill_reddit_general",
  "name": "reddit",
  "description": "Search Reddit for real community opinions on any topic. Use only when explicitly invoked with /reddit.",
  "creatorType": "user",
  "updatedAt": "2026-06-03T00:00:00.000Z",
  "enabled": true
}
```

**4. Restart Claude Code.**

## Contents

- `SKILL.md` — the skill definition (instructions, subreddit targeting logic, tool usage rules, token budget)

## Notes

- The skill is **never auto-triggered** — it only runs when you explicitly call `/reddit`. This is intentional.
- PullPush archive ends ~May 2025. Reddit Buddy covers everything after. Both are used on every query.
- No Reddit account or API credentials needed.
