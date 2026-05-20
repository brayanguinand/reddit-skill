---
name: reddit
description: Search Reddit for real community opinions, experiences, and recommendations on any topic. Use ONLY when the user explicitly asks to "search on Reddit", "use the Reddit skill", or "/reddit". Never activate automatically — this is a deliberate research action. Returns synthesized community views from actual Reddit threads, not web search results. Especially useful for product recommendations, software opinions, local knowledge, lifestyle questions, and anything where peer experience beats editorial content.
argument-hint: <query or question>
allowed-tools: ["mcp__pullpush__search_submissions", "mcp__pullpush__search_comments", "mcp__reddit-buddy__search_reddit", "mcp__reddit-buddy__get_post_details", "Bash"]
---

# Reddit Skill — Community Research Tool

The user asked: **$ARGUMENTS**

## How to use this skill

You have direct access to Reddit via two MCP tools:
- **`mcp__pullpush__search_submissions`** — searches the PullPush archive (posts up to ~May 2025). Fast, no rate limit issues. Use this first.
- **`mcp__pullpush__search_comments`** — fetches comments for a specific post by `link_id`.
- **`mcp__reddit-buddy__search_reddit`** — searches Reddit in real time (post-May 2025). Use only if recency matters.
- **`mcp__reddit-buddy__get_post_details`** — gets full post + comments in real time.

## Step 1 — Identify target subreddits

Do NOT search `r/all` — it returns noise. Think about which specific subreddits would have the most relevant community for this query.

Examples:
- Accounting software in France → `r/comptabilite`, `r/france`, `r/entrepreneur`  
- Best Python web framework → `r/Python`, `r/django`, `r/webdev`
- Coffee grinder recommendations → `r/Coffee`, `r/espresso`
- Moving to Berlin → `r/berlin`, `r/germany`, `r/expats`

Pick 1-3 subreddits max. If unsure, start broad (country or topic subreddit), then add a niche one.

## Step 2 — Search posts (token budget: tight)

Run `mcp__pullpush__search_submissions` for each subreddit:
- `limit`: 15 (over-fetch to filter)
- `sort_type`: `num_comments` (most discussed = most useful)
- Skip posts with 0 or 1 comment

**Select the 3-5 most relevant posts** based on title relevance and comment count. Do not fetch comments for all posts.

If the query is about something post-2025, add one `mcp__reddit-buddy__search_reddit` call.

## Step 3 — Fetch comments selectively

Only fetch comments for the **2-3 most promising posts**. Use:
- `mcp__pullpush__search_comments` with `link_id=<post_id>`, `limit=30`
- Or `mcp__reddit-buddy__get_post_details` for recent posts

From the comments, extract:
- Specific product/tool/place names mentioned with positive or negative context
- Recurring recommendations (if X appears in 3+ comments independently, it's a strong signal)
- Caveats, dealbreakers, or "avoid X because..." patterns

## Step 4 — Synthesize and respond

**Do not dump raw Reddit content.** Synthesize:

1. **What the community recommends** — top picks with brief reasoning
2. **What to avoid / caveats** — recurring warnings
3. **Nuance** — if there are diverging opinions, explain why
4. **Sources** — include 2-3 Reddit thread URLs so the user can dive deeper

Keep the response concise. If the topic needs a list, use one. Match the language of the user's question.

## Token efficiency rules

- Max 15 posts fetched per search call
- Comments only for 2-3 posts max
- If pullpush returns strong results → skip reddit-buddy
- If a post has few comments and a low score, skip it
- Do not repeat raw comment text — extract the insight, discard the rest

## Fallback

If pullpush returns nothing relevant: try a different subreddit or rephrase the query. If reddit-buddy also returns nothing: tell the user the topic has low Reddit coverage and suggest alternatives.
