---
name: reddit
description: Search Reddit for real community opinions, experiences, and recommendations on any topic. Use ONLY when the user explicitly asks to "search on Reddit", "use the Reddit skill", or "/reddit". Never activate automatically — this is a deliberate research action. Returns synthesized community views from actual Reddit threads, not web search results. Especially useful for product recommendations, software opinions, local knowledge, lifestyle questions, and anything where peer experience beats editorial content.
allowed-tools: ["mcp__pullpush__search_submissions", "mcp__pullpush__search_comments", "mcp__reddit-buddy__search_reddit", "mcp__reddit-buddy__browse_subreddit", "mcp__reddit-buddy__get_post_details"]
---

# Reddit Skill — Community Research Tool

The user asked: **$ARGUMENTS**

## Tools available

- **`mcp__pullpush__search_submissions`** — PullPush archive (posts up to ~May 2025). Rate limit: 15 req/min soft, 30 hard, 1000/hour. Use for historical depth.
- **`mcp__pullpush__search_comments`** — searches comments by keyword across the archive. Use only for broad comment-level searches (no post filtering by ID).
- **`mcp__reddit-buddy__search_reddit`** — real-time Reddit search (covers everything, including the last 13 months missing from PullPush). Use for recent content or as complement.
- **`mcp__reddit-buddy__browse_subreddit`** — fetches top/hot/new posts from a subreddit. Best for "what's the best X" queries: use `sort=top&time=year` to get community-validated content directly.
- **`mcp__reddit-buddy__get_post_details`** — fetches a full post + comments by ID or URL. Always pass `subreddit` alongside `post_id` to save one API call.

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

## Step 2 — Search posts (run in parallel)

**Always run subreddit searches in parallel** — fire all calls in a single message, not sequentially.

**For archive content (pre-May 2025):** use `mcp__pullpush__search_submissions` per subreddit:
- `size: 15` (over-fetch to filter)
- `sort_type: num_comments` (most discussed = most signal)
- `num_comments: ">3"` (server-side filter — don't fetch noise)
- `score: ">2"` if you want only positively received posts
- Use `title` param instead of `q` for cleaner title-only matching on recommendation queries

**For recent content (post-May 2025 — last 13 months):** always add at least one `mcp__reddit-buddy__search_reddit` call:
- `sort: "comments"` (equivalent of most-discussed)
- `time: "year"` for recent but not too narrow

**Select the 3-5 most relevant posts** across both sources based on title relevance and comment count.

## Step 3 — Fetch comments selectively

Only fetch comments for the **1-2 most promising posts**. Use `mcp__reddit-buddy__get_post_details`:
- Always pass both `post_id` AND `subreddit` (saves one API call vs post_id alone)
- `comment_sort: "best"` (default — keep it)
- `max_top_comments: 10` (default is 5, too few for research)
- `comment_limit: 20`

Do NOT use `mcp__pullpush__search_comments` to fetch comments for a specific post — it has no post ID filter and will return unrelated data. Use it only for keyword-based comment searches across a whole subreddit.

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
