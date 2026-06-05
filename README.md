# Reddit Skill for Claude Code

A Claude Code skill that gives you real Reddit search from any session — no Reddit API credentials required.

Reddit blocks web crawlers and `site:reddit.com` doesn't work in search. This skill bypasses that by using MCP servers that talk directly to Reddit's data. You get actual threads and comments, not SEO aggregators.

## Usage

Once installed, invoke it with:

```
/reddit <your question>
```

Examples:
```
/reddit best accounting software for freelancers in France
/reddit mechanical keyboard under $150
/reddit is Lisbon still affordable for expats in 2025
/reddit airfryer vs convection oven which is actually better
/reddit FastAPI vs Django for a REST API in 2026
```

The skill targets the right subreddits automatically, fetches the most-discussed or highest-scored threads, reads the top comments, and returns a synthesized answer with source links — not a raw data dump.

## How it works

Three MCP servers with complementary coverage:

```
2005 ──────────── May 2025   →  PullPush (archive)
May 2025 ────────────── now  →  Arctic Shift (main source, ~24-48h indexing lag)
                              →  Reddit Buddy (real-time, frequently blocked by Reddit)
```

- **PullPush MCP** — searches the PullPush Reddit archive (coverage up to ~May 2025). Rate limit: 15 req/min soft, 30 hard, 1000/hour. Used for pre-2025 content.
- **Arctic Shift MCP** — main source for everything from May 2025 to today. Full engagement data (scores, comment counts) for posts older than ~48h. Very recent posts (< 24-48h) are indexed but show `score: 1, comments: 0` until engagement data catches up.
- **Reddit Buddy MCP** — hits Reddit's live endpoints in real time. Currently blocked by Reddit on most queries (access forbidden errors). Included as a last resort for the last 24-48h window; the skill tries it once and skips gracefully if blocked.

The skill picks 1-3 relevant subreddits per query (never `r/all`), runs PullPush + Arctic Shift in parallel, reads comments on the 1-2 most relevant posts via Arctic Shift's comment tree API, and synthesizes.

### Sort strategy

Arctic Shift supports three sort modes the skill uses selectively:
- `score` — for "best X", "X vs Y", "should I use X" queries (evergreen recommendations)
- `num_comments` — for advice threads and open-ended discussions
- `date_desc` + `after: <epoch>` — for breaking topics from the last few days

### Known limitations

- **PullPush `num_comments` server-side filter is broken** — the skill filters manually after fetching.
- **PullPush `after` only accepts Unix epoch timestamps** — relative formats like `"1y"` cause a 400 error.
- **Arctic Shift `query` requires `subreddit` or `author`** — query-only calls return a 400 error.
- **Reddit Buddy is frequently blocked** — don't rely on it for production queries.
- **French subreddits** (r/france, r/AskFrance) have low post density on business/tech topics — the skill automatically runs an English-language equivalent subreddit in parallel.

## Prerequisites

Install all three MCP servers globally:

```bash
npm install -g pullpush-mcp
npm install -g reddit-mcp-buddy
npm install -g arctic-shift-mcp
```

## Installation

**1. Add the MCPs at user scope** (makes them available in every Claude Code session):

```bash
claude mcp add pullpush /opt/homebrew/bin/pullpush-mcp --scope user
claude mcp add reddit-buddy /opt/homebrew/bin/reddit-mcp-buddy --scope user
claude mcp add arctic-shift /opt/homebrew/bin/arctic-shift-mcp --scope user
```

Adjust the binary paths if npm installed them elsewhere (`which pullpush-mcp` to check).

**2. Copy `SKILL.md` to your Claude Desktop skills folder:**

On macOS:
```
~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/<plugin-id>/<session-id>/skills/reddit/SKILL.md
```

The easiest way is to use the Claude Code skill-creator skill, or manually drop the file into the `skills/` directory alongside your other skills.

**3. Register the skill in `manifest.json`** in the same folder:

```json
{
  "skillId": "skill_reddit_general",
  "name": "reddit",
  "description": "Search Reddit for real community opinions on any topic. Use only when explicitly invoked with /reddit.",
  "creatorType": "user",
  "updatedAt": "2026-06-05T00:00:00.000Z",
  "enabled": true
}
```

**4. Restart Claude Code.**

## Contents

- `SKILL.md` — the skill definition (instructions, subreddit targeting logic, tool usage rules, token budget)

## Notes

- The skill is **never auto-triggered** — it only runs when you explicitly call `/reddit`. This is intentional.
- No Reddit account or API credentials needed.
- PullPush and Arctic Shift are independent archives — they don't call Reddit at query time.
- Reddit Buddy reads Reddit's public `.json` endpoints (same data your browser loads). These endpoints predate the API and Reddit can't easily close them without breaking their own site — but Reddit does rate-limit and block them intermittently.
