---
name: mmk-threads-posts
description: Get recent Threads posts with optional insights and date filtering. Triggers on "threads posts", "my posts", "recent posts", "show posts".
allowed-tools: Bash(mmk *)
---

# MMK Threads — Posts

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-threads`](../mmk-threads/SKILL.md) — includes Pagination guidance.

Always use `-o json` when parsing results or composing with other commands.

---

## posts — Get my recent posts

```bash
mmk threads posts -o json
mmk threads posts --limit 10 --insights -o json
mmk threads posts --since 2025-01-01 --until 2025-06-30 -o json
mmk threads posts --after <cursor> -o json
```

**Optional flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | `25` | Number of posts per page |
| `--insights` | `false` | Include views/likes/replies counts |
| `--since` | | Start date (Unix timestamp or ISO 8601) |
| `--until` | | End date (Unix timestamp or ISO 8601) |
| `--after` | | Pagination cursor for next page |

Output includes a "Next page" hint with the cursor when more results are available.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-threads](../mmk-threads/SKILL.md) — Pagination, all Threads commands overview
- [mmk-threads-insights](../mmk-threads-insights/SKILL.md) — Account analytics
- [mmk-threads-replies](../mmk-threads-replies/SKILL.md) — Post replies/conversation
