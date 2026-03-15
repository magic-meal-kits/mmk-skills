---
name: mmk-threads-insights
description: Get Threads account analytics and metrics with breakdowns. Triggers on "threads insights", "threads analytics", "account metrics", "threads stats", "engagement metrics".
allowed-tools: Bash(mmk *)
---

# MMK Threads — Insights

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-threads`](../mmk-threads/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## insights — Account analytics

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

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-threads](../mmk-threads/SKILL.md) — All Threads commands overview
- [mmk-threads-posts](../mmk-threads-posts/SKILL.md) — Get recent posts with insights
