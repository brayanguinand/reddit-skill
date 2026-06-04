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

### Arctic Shift (archive up to ~February 2026)
- **`mcp__arctic-shift__search_submissions`** — keyword search on posts, with `sort_by: num_comments` or `score` (client-side). Covers 9 more months than PullPush.
- **`mcp__arctic-shift__search_comments`** — keyword search on comments with **working `link_id` filter** (get comments from a specific post).
- **`mcp__arctic-shift__get_post_comments`** — full nested comment tree for a post by ID.

### Reddit Buddy (real-time, last ~4 months)
- **`mcp__reddit-buddy__search_reddit`** — live Reddit search. Use only for content from March 2026 onwards. May be unavailable if Reddit blocks the IP.
- **`mcp__reddit-buddy__browse_subreddit`** — top/hot/new posts from a subreddit. Use `sort=top&time=year` for "best of" queries.
- **`mcp__reddit-buddy__get_post_details`** — full post + comments. Always pass `subreddit` + `post_id` together (saves 1 API call).

## Step 1 — Identify target subreddits

Do NOT search `r/all` — it returns noise. Pick 1-3 specific subreddits with the most relevant community for the query.

Examples:
- Accounting software in France → `r/comptabilite`, `r/france`, `r/entrepreneur`
- Best Python web framework → `r/Python`, `r/django`, `r/webdev`
- Coffee grinder recommendations → `r/Coffee`, `r/espresso`
- Moving to Berlin → `r/berlin`, `r/germany`, `r/expats`

If the query is in French or another language, also consider subreddits in that language.

**For "best of" queries** (recommendations, rankings): prefer `browse_subreddit` with `sort=top&time=year` — it gives directly the most upvoted content without keyword guessing.

**For targeted queries**: check if the subreddit uses flairs and filter by the most relevant one. Examples:
- `r/personalfinance` → flair "Investing", "Debt", "Retirement"
- `r/Python` → flair "Discussion", "Tutorial"
- `r/france` → flair "Société", "Économie"
Use the `flair` parameter in `mcp__reddit-buddy__search_reddit` when a specific flair would sharply reduce noise.

## Step 2 — Search posts

Each source covers an **exclusive, non-overlapping window**. Do NOT query two sources for the same period — they have the same data.

```
2005 ──────────── May 2025   →  PullPush only
         May 2025 ── Feb 2026  →  Arctic Shift only
                  Feb 2026 ── now  →  Reddit Buddy only
```

**For most queries: run PullPush + Arctic Shift in parallel** (they cover complementary windows, not the same content). Add Reddit Buddy only if recency matters.

**PullPush** (2005 – May 2025):
- `mcp__pullpush__search_submissions` with `size: 15`, `sort_type: num_comments`, `num_comments: ">3"`
- Use `title` param for title-only matching (cleaner than `q`)

**Arctic Shift** (May 2025 – February 2026):
- `mcp__arctic-shift__search_submissions` with `limit: 15`, `sort_by: num_comments`
- Supports `link_flair_text` for flair filtering

**Reddit Buddy** (February 2026 – today) — only if recency matters:
- `mcp__reddit-buddy__search_reddit` with `sort: "comments"`, `time: "year"`
- May be unavailable (Reddit IP blocking). If down: proceed without it and note the gap to the user.

**Select the 3-5 most relevant posts** across all sources based on title relevance and comment count.

## Step 3 — Fetch comments selectively

Only fetch comments for the **1-2 most promising posts**.

**For archive posts (pre-Feb 2026):** use `mcp__arctic-shift__get_post_comments`:
- `link_id: <post_id>`, `limit: 50`
- Returns nested tree sorted by score — best option for archive posts

**For recent posts (post-Feb 2026):** use `mcp__reddit-buddy__get_post_details`:
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

- Max `size: 15` per search call, with server-side filters (`num_comments`, `score`) to reduce noise at the source
- Comments for 1-2 posts max via `get_post_details`, always with `subreddit` param
- Never quote raw comments — extract the insight only
- Response: plain prose or short list, no tables, no headers per point

## Fallback

If pullpush returns nothing relevant: try a different subreddit, rephrase the query, or switch to `browse_subreddit`. If reddit-buddy also returns nothing: tell the user the topic has low Reddit coverage and suggest alternatives.
