# Reddit Skill for Claude Code

A Claude Code skill that gives you real Reddit search from any session — no Reddit API credentials required.

Reddit blocks web crawlers and `site:reddit.com` doesn't work in search. This skill bypasses that by using two MCP servers that talk directly to Reddit's data: one for the archive (fast, great coverage), one for real-time results. You get actual threads and comments, not SEO aggregators.

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

- **PullPush MCP** — searches the PullPush Reddit archive (coverage up to ~May 2025). Used first for most queries.
- **Reddit Buddy MCP** — searches Reddit in real time. Used when recency matters (post-2025 content).
- The skill picks 1-3 relevant subreddits per query (never `r/all`), fetches up to 15 posts, reads comments on the 2-3 most relevant ones, and synthesizes. Token-efficient by design.

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
  "updatedAt": "2026-05-20T00:00:00.000Z",
  "enabled": true
}
```

**4. Restart Claude Code.**

## Contents

- `SKILL.md` — the skill definition (instructions, subreddit targeting logic, token budget rules)

## Notes

- The skill is **never auto-triggered** — it only runs when you explicitly call `/reddit`. This is intentional.
- PullPush archive coverage ends around May 2025. For anything more recent, the skill falls back to Reddit Buddy automatically.
- No Reddit account or API credentials needed.
