---
name: mmk-notion-user
description: Manage Notion workspace users — get user details, list all users. Triggers on "get user", "user details", "list users", "workspace users", "who is user".
allowed-tools: Bash(mmk *)
---

# MMK Notion — User Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## get — Get user details

```bash
mmk notion user get --user-id <user-id> -o json
```

**Required:** `--user-id`

---

## list — List workspace users

```bash
mmk notion user list -o json
mmk notion user list --limit 50 -o json
```

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | — | Maximum number of users to return |
| `--cursor` | — | Pagination cursor for next page of results |

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-people](../mmk-notion-people/SKILL.md) — List members/guests, guest page access
