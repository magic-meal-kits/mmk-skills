---
name: mmk-threads
description: This skill should be used when the user asks to view Threads posts, check account insights/analytics, get replies to a post, or any Threads (Meta) social media operation using the mmk CLI. Triggers on phrases like "threads posts", "threads insights", "threads replies", "threads analytics", "my posts", "account metrics", "post conversation".
allowed-tools: Bash(mmk *)
---

# MMK Threads

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

Access Threads (Meta) account data — posts, insights, and replies — using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `posts` | Get recent posts with optional insights and date filtering | [`mmk-threads-posts`](../mmk-threads-posts/SKILL.md) |
| `insights` | Account analytics with metric and breakdown options | [`mmk-threads-insights`](../mmk-threads-insights/SKILL.md) |
| `replies` | Get replies/conversation for a specific post | [`mmk-threads-replies`](../mmk-threads-replies/SKILL.md) |

## Pagination

Both `posts` and `replies` support cursor-based pagination via `--after`. When more results are available, the output includes a "Next page" hint with the cursor value to use.
