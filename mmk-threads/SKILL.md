---
name: mmk-threads
description: This skill should be used when the user asks to view Threads posts, check account insights/analytics, get replies to a post, or any Threads (Meta) social media operation using the mmk CLI. Triggers on phrases like "threads posts", "threads insights", "threads replies", "threads analytics", "my posts", "account metrics", "post conversation".
allowed-tools: Bash(mmk *)
---

# MMK Threads

Access Threads (Meta) account data — posts, insights, and replies — using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

---

## Commands

### posts — Get my recent posts

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

### insights — Account analytics

```bash
mmk threads insights -o json
mmk threads insights --since 2025-01-01 --until 2025-06-30 -o json
mmk threads insights --breakdown country -o json
mmk threads insights --metric views,likes -o json
```

**Optional flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--metric` | `views,likes,replies,reposts,quotes,followers_count` | Comma-separated metrics |
| `--since` | | Start date |
| `--until` | | End date |
| `--breakdown` | | Breakdown type: `country`, `city`, `age`, `gender` |

### replies — Get replies/conversation for a post

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

## Pagination

Both `posts` and `replies` support cursor-based pagination via `--after`. When more results are available, the output includes a "Next page" hint with the cursor value to use.
