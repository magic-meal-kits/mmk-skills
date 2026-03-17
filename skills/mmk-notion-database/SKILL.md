---
name: mmk-notion-database
description: Manage Notion databases — show property schema, query with filters/sorts, trigger AI summary generation, insert/update/upsert/delete records. Triggers on "database schema", "AI summary", "database properties", "detect AI", "ai-summary", "database structure", "query database", "filter", "sort", "database query", "insert records", "update records", "upsert", "delete pages", "bulk insert", "bulk update", "bulk delete".
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

## insert — Bulk insert records into a Notion database

```bash
mmk notion database insert --database-id <db> --data '{"Name":"Test","Status":"Active"}' -o json
mmk notion database insert --database-id <db> --data '[{"Name":"A"},{"Name":"B"}]' -o json
mmk notion database insert --database-id <db> --file records.json -o json
mmk notion database insert --database-id <db> --set Name=Test --set Status=Active -o json
```

**Required:** `--database-id`, one of `--data`, `--file`, `--set`, or piped stdin
**Optional:**

| Flag | Description |
|------|-------------|
| `--show-schema` | Show database schema and exit |
| `--icon-emoji` | Emoji icon for new pages |
| `--template-id` | Template page ID for new pages |

---

## update — Bulk update existing Notion pages

```bash
mmk notion database update --data '{"page_id":"abc-123","Status":"Done"}' -o json
mmk notion database update --data '[{"page_id":"abc","Status":"Done"},{"page_id":"def","Status":"Active"}]' -o json
mmk notion database update --page-id abc-123 --set Status=Done --set Priority=High -o json
mmk notion database update --file updates.json -o json
```

**Required:** one of `--data` (each object must include `page_id`), `--file`, or `--page-id` + `--set`, or piped stdin

---

## upsert — Bulk upsert records in a Notion database

```bash
mmk notion database upsert --database-id <db> --lookup Name --data '[{"Name":"Test","Status":"Done"}]' -o json
mmk notion database upsert --database-id <db> --lookup Name,Email --file records.json -o json
mmk notion database upsert --database-id <db> --lookup Name --update-properties Status,Priority --set Name=Test --set Status=Done -o json
```

**Required:** `--database-id`, `--lookup` (property names for matching, repeatable or comma-separated), one of `--data`, `--file`, `--set`, or piped stdin
**Optional:**

| Flag | Description |
|------|-------------|
| `--update-properties` | Property names to update on match (optional — updates all if omitted) |
| `--show-schema` | Show database schema and exit |
| `--icon-emoji` | Emoji icon for new pages |
| `--template-id` | Template page ID for new pages |

---

## delete — Bulk delete (archive) Notion pages

```bash
mmk notion database delete --page-id abc-123 --page-id def-456 -o json
mmk notion database delete --data '["abc-123","def-456"]' -o json
mmk notion database delete --file page_ids.json -o json
```

**Required:** one of `--page-id` (repeatable), `--data` (JSON array of ID strings), `--file`, or piped stdin

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
- [mmk-notion-database-ai-summary](../mmk-notion-database-ai-summary/SKILL.md) — Bulk AI summary generation (recipe)
- [mmk-paymint-notion-invoice](../mmk-paymint-notion-invoice/SKILL.md) — Batch Paymint invoice from Notion database (recipe)
