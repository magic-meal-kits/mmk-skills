---
name: mmk-threads-replies
description: Get replies and conversation for a Threads post. Triggers on "threads replies", "post conversation", "show replies", "post replies".
allowed-tools: Bash(mmk *)
---

# MMK Threads — Replies

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-threads`](../mmk-threads/SKILL.md) — includes Pagination guidance.

Always use `-o json` when parsing results or composing with other commands.

---

## replies — Get replies/conversation for a post

```bash
mmk threads replies <media_id> -o json
mmk threads replies 12345678 --limit 20 -o json
mmk threads replies 12345678 --reverse -o json
mmk threads replies 12345678 --after <cursor> -o json
```

**Positional argument:** `<media_id>` (required — the post ID)

**Optional flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | `50` | Max replies per page |
| `--reverse` | `false` | Oldest first |
| `--after` | | Pagination cursor for next page |

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-threads](../mmk-threads/SKILL.md) — Pagination, all Threads commands overview
- [mmk-threads-posts](../mmk-threads-posts/SKILL.md) — Get recent posts
