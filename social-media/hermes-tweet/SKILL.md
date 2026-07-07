---
name: hermes-tweet
description: "Use Hermes Tweet for X/Twitter research, drafting, monitoring, follower exports, and explicitly approved account actions."
version: 0.1.6
author: Xquik-dev
metadata:
  hermes:
    tags: [Social Media, X, Twitter, Automation, Monitoring]
---

# Hermes Tweet

Use Hermes Tweet when a Hermes Agent user needs X/Twitter research, drafting,
monitoring, follower exports, or approved account actions.

## When to Use

- User asks to draft, improve, or review a tweet or X thread.
- User asks for public X/Twitter context from posts, users, timelines, replies,
  search, or followers.
- User asks to monitor campaign replies, mentions, competitors, or social
  signals.
- User explicitly approves a state-changing X/Twitter action.

## Prerequisites

- Hermes Tweet installed from `Xquik-dev/hermes-tweet`.
- `XQUIK_API_KEY` configured in the Hermes runtime environment before
  authenticated reads.
- Action enablement configured before publishing, replying, liking, reposting,
  following, or deleting.

## Tool Routing

| Tool | Use |
| --- | --- |
| `tweet_explore` | Offline planning, examples, and draft shaping. |
| `tweet_read` | Public X/Twitter reads for live evidence. |
| `tweet_action` | State-changing actions after explicit user approval. |

## Procedure

1. Classify the request as draft, read, monitor, export, or action.
2. Start with `tweet_explore` when no live read is needed.
3. Use `tweet_read` when the user needs current public X/Twitter evidence.
4. Summarize evidence with identifiers or links returned by the tool.
5. Ask for explicit approval before `tweet_action`.
6. Confirm the action result after the tool returns.

## Safety Rules

- Never publish, reply, repost, like, follow, or delete without explicit user
  approval.
- Never put credentials, cookies, tokens, or API keys in repository files.
- Do not invent live metrics when reads are unavailable.
- Keep generated posts concise and matched to the user's voice.

Repository: https://github.com/Xquik-dev/hermes-tweet
