---
name: reddit
description: Search Reddit for real community opinions, experiences, and recommendations on any topic. Use ONLY when the user explicitly asks to "search on Reddit", "use the Reddit skill", or "/reddit". Never activate automatically — this is a deliberate research action. Returns synthesized community views from actual Reddit threads, not web search results. Especially useful for product recommendations, software opinions, local knowledge, lifestyle questions, and anything where peer experience beats editorial content.
allowed-tools: ["mcp__pullpush__search_submissions", "mcp__pullpush__search_comments", "mcp__reddit-buddy__search_reddit", "mcp__reddit-buddy__browse_subreddit", "mcp__reddit-buddy__get_post_details", "mcp__arctic-shift__search_submissions", "mcp__arctic-shift__search_comments", "mcp__arctic-shift__get_post_comments"]
---

# Reddit Skill — Community Research Tool

The user asked: **$ARGUMENTS**

## Tools available

### PullPush (archive up to ~May 2025)
- **`mcp__pullpush__search_submissions`** — keyword search on posts. Rate limit: 15 req/min soft, 30 hard, 1000/hour.
- **`mcp__pullpush__search_comments`** — keyword search on comments. No post ID filter (use arctic-shift for that).

### Arctic Shift (main source — covers today, full engagement data up to ~3-4 weeks ago)
- **`mcp__arctic-shift__search_submissions`** — keyword search on posts. **Requires `subreddit` or `author` alongside `query`** — query-only calls return a 400 error.
- **`mcp__arctic-shift__search_comments`** — keyword search on comments with **working `link_id` filter**.
- **`mcp__arctic-shift__get_post_comments`** — full nested comment tree for a post by ID. Best option for fetching comments.

### Reddit Buddy (real-time — currently unreliable)
- **`mcp__reddit-buddy__search_reddit`** and **`mcp__reddit-buddy__browse_subreddit`** — live Reddit access. **Frequently blocked by Reddit (access forbidden errors).** Try once; if it fails, skip and note the gap to the user. Do not retry.
- **`mcp__reddit-buddy__get_post_details`** — full post + comments. Always pass both `post_id` AND `subreddit`.

## Step 1 — Identify target subreddits

Do NOT search `r/all` — it returns noise. Pick 1-3 specific subreddits with the most relevant community for the query.

Examples:
- Accounting software in France → `r/comptabilite`, `r/france`, `r/entrepreneur`
- Best Python web framework → `r/Python`, `r/django`, `r/webdev`
- Coffee grinder recommendations → `r/Coffee`, `r/espresso`
- Moving to Berlin → `r/berlin`, `r/germany`, `r/expats`

If the query is in French or another language, also consider the English-language equivalent subreddit — French subreddits (`r/france`, `r/AskFrance`) have much lower post density on business/tech topics. Always run both in parallel.

**For "best of" queries** (recommendations, rankings): use `arctic-shift__search_submissions` with `sort_by: score` — it returns the most upvoted posts directly.

**For targeted queries**: filter by flair using `link_flair_text` in `mcp__arctic-shift__search_submissions` to reduce noise. Examples:
- `r/personalfinance` → flair "Investing", "Debt"
- `r/Python` → flair "Discussion"
- `r/france` → flair "Société", "Économie"

## Step 2 — Search posts

```
2005 ──────────── May 2025   →  PullPush (archive)
May 2025 ────────────── now  →  Arctic Shift (full engagement data up to ~3-4 weeks ago; partial after)
                              →  Reddit Buddy (real-time, but frequently blocked — try once only)
```

**For most queries: run PullPush + Arctic Shift in parallel.** They cover different time windows.

**PullPush** (2005 – May 2025):
- `mcp__pullpush__search_submissions` with `size: 15`, `sort_type: num_comments`
- Use `title` param for title-only matching (cleaner than `q`)
- ⚠️ The `num_comments` server-side filter is broken — it does not filter reliably. After fetching, **manually skip posts with fewer than 3 comments**.
- ⚠️ The `after` parameter only accepts **Unix epoch timestamps** — relative formats like "1y" or "30d" cause a 400 error. To restrict to recent posts, compute the epoch: e.g., June 2024 = `1717545600`. If unsure, skip `after` — `sort_type: num_comments` naturally surfaces recent high-engagement posts.

**Arctic Shift** (May 2025 – today):
- `mcp__arctic-shift__search_submissions` with `limit: 15`
- **Always pass `subreddit` (or `author`) alongside `query` — query-only calls return a 400 error**
- For recommendations, opinions, comparisons (evergreen): use `sort_by: score` — surfaces best posts across all time. Strongly preferred for "best X", "X vs Y", "should I use X" queries.
- For discussions, advice threads: use `sort_by: num_comments` — surfaces most debated posts.
- For recent/breaking topics (last few days): use `sort_by: date_desc` with `after: <epoch>` — engagement data for posts older than ~48h is reliable; very recent posts (< 24-48h) show `score: 1, comments: 0` and should be used for content only, not engagement signal.

**Reddit Buddy** (real-time) — only if recency matters AND content is from the last few days:
- Try `mcp__reddit-buddy__search_reddit` once with `sort: "comments"`, `time: "month"`
- If it returns any error (blocked, forbidden, failed), skip entirely and note the gap to the user. Do not retry.

**Select the 3-5 most relevant posts** across all sources based on title relevance and comment count.

## Step 3 — Fetch comments selectively

Only fetch comments for the **1-2 most promising posts**.

**For all archive posts (use Arctic Shift — works for any post):** use `mcp__arctic-shift__get_post_comments`:
- `link_id: <post_id>`, `limit: 50`
- Returns nested tree sorted by score — best option

**For very recent posts only (last few days, not yet in Arctic Shift):** use `mcp__reddit-buddy__get_post_details`:
- Always pass both `post_id` AND `subreddit` (saves one API call)
- `comment_sort: "best"`, `max_top_comments: 10`, `comment_limit: 20`

Do NOT use `mcp__pullpush__search_comments` to fetch comments for a specific post — it has no post ID filter.

Extract only: recurring recommendations, named products/places/tools, explicit warnings. Discard everything else.

## Step 4 — Synthesize and respond

**Do not dump raw Reddit content.** Synthesize:

1. **What the community recommends** — top picks with brief reasoning
2. **What to avoid / caveats** — recurring warnings
3. **Nuance** — if there are diverging opinions, explain why
4. **Sources** — include 2-3 Reddit thread URLs so the user can dive deeper

Keep the response concise. If the topic needs a list, use one. Match the language of the user's question.

## Token efficiency rules

- Max `size: 15` per PullPush call, `limit: 15` per Arctic Shift call
- After PullPush fetch, manually discard posts with fewer than 3 comments (server-side filter unreliable)
- Comments for 1-2 posts max via `arctic-shift__get_post_comments`
- Never quote raw comments — extract the insight only
- Response: plain prose or short list, no tables, no headers per point

## Fallback

If PullPush + Arctic Shift return nothing relevant: try a different subreddit or rephrase the query. If the query is in French, also try the English-language equivalent subreddit. If all sources fail: tell the user the topic has low Reddit coverage and suggest alternatives (specialized forums, Stack Overflow, dedicated communities).
