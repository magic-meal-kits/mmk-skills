---
name: mmk-notion-database
description: Manage Notion databases — show property schema, query with filters/sorts, trigger AI summary generation. Triggers on "database schema", "AI summary", "database properties", "detect AI", "ai-summary", "database structure", "query database", "filter", "sort", "database query".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Database Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## schema — Show database property schema

```bash
mmk notion database schema --database-id <db-uuid> -o json
mmk notion database schema --database-id <db-uuid> --detect-ai --page-id <page-uuid> -o json
```

**Required:** `--database-id`
**Optional:** `--detect-ai` (bool), `--page-id` (required with `--detect-ai` — accepts UUID or full Notion URL), `--enhanced` (bool — outputs LLM-friendly schema with select options and valid filter operators)

When `--detect-ai` is set, output includes `property-id` and `workflow-id` values needed for the `ai-summary` command.

---

## query — Query a Notion database with filters, sorts, and pagination

```bash
mmk notion database query --database-id <db-uuid> -o json
mmk notion database query --database-id <db> --filter "Status=Active" --sort "CreatedDate:desc" --limit 10 -o json
mmk notion database query --database-id <db> --filter "Status=Active" --filter "Done=false" --properties "Name,Status,Price" -o json
mmk notion database query --database-id <db> --cursor "eyJsYXN0..." -o json
```

**Required:** `--database-id`
**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--filter` | | Filter: `Key=Value` or `Key:operator=Value` (repeatable, AND logic) |
| `--sort` | | Sort: `Property:asc` or `Property:desc` (repeatable) |
| `--limit` | | Max results per page (max 100) |
| `--properties` | | Property names to return (comma-separated) |
| `--cursor` | | Pagination cursor for next page |

Use `schema --enhanced` to discover available properties, types, and valid operators.

---

## ai-summary — Trigger AI summary generation for database items

```bash
mmk notion database ai-summary --database-id <db-uuid> --page-id <page-1> --page-id <page-2> --property-id "prop_id" -o json
```

**Required:** `--database-id`, `--page-id` (string slice, repeatable), `--property-id`
**Optional:** `--workflow-id`

**Limits:** Max 10 page IDs per execution. Use `schema --detect-ai` first to discover `--property-id` and `--workflow-id`.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-page](../mmk-notion-page/SKILL.md) — Page invite, publish, and config
