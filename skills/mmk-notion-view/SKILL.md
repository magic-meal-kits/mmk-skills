---
name: mmk-notion-view
description: Manage Notion database views — list, get, create, update, delete. Triggers on "database view", "notion view", "list views", "create view", "update view", "delete view", "get view".
allowed-tools: Bash(mmk *)
---

# MMK Notion — View Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## list — List all views for a database

```bash
mmk notion view list --database-id <db-id> -o json
```

**Required:** `--database-id`

---

## get — Get a specific view by ID

```bash
mmk notion view get <view_id> -o json
```

**Required:** `<view_id>` (positional)

---

## create — Create a new view for a database

```bash
mmk notion view create --database-id <db-id> --data-source-id <ds-id> --name "My View" --type table -o json
mmk notion view create --database-id <db-id> --data '{"data_source_id":"...","name":"My View","type":"board"}' -o json
```

**Required:** `--database-id`, one of `--data-source-id` or `--data`

**Optional:**

| Flag | Description |
|------|-------------|
| `--name` | View name |
| `--type` | View type: `table`, `board`, `timeline`, `calendar`, `list`, `gallery` |
| `--data` | Raw JSON request body |

Use `mmk notion view get <view_id> -o json` to retrieve the `data_source_id` from an existing view.

---

## update — Update an existing view

```bash
mmk notion view update <view_id> --data '{"name":"New Name"}' -o json
```

**Required:** `<view_id>` (positional), `--data`

---

## delete — Delete a view

```bash
mmk notion view delete <view_id> -o json
```

**Required:** `<view_id>` (positional)

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-database](../mmk-notion-database/SKILL.md) — Database schema, query, insert, update, upsert, delete, AI summary
