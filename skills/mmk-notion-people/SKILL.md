---
name: mmk-notion-people
description: Manage Notion workspace people — list members/guests/owners, show guest page access. Triggers on "list members", "workspace members", "list guests", "guest pages", "who has access".
allowed-tools: Bash(mmk *)
---

# MMK Notion — People Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## list — List workspace members and guests

```bash
mmk notion people list -o json
mmk notion people list --type guest -o json
```

**Optional:** `--type` — `all` (default), `member`, `guest`, or `owner`

---

## guest-pages — Show guest page access

```bash
mmk notion people guest-pages --user-id <uuid> -o json
mmk notion people guest-pages --user-id id1 --user-id id2 -o json
```

**Required:** `--user-id` (string slice — supports multiple user IDs)

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-page](../mmk-notion-page/SKILL.md) — Page-level invite and permissions
- [mmk-notion-workspace](../mmk-notion-workspace/SKILL.md) — Workspace invite and remove
