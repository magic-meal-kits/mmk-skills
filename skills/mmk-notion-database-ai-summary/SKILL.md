---
name: mmk-notion-database-ai-summary
description: Bulk AI summary generation across all pages in a Notion database. Triggers on "generate AI summaries for database", "bulk AI summary", "batch AI summary", "run AI summary across database".
disable-model-invocation: true
allowed-tools: Bash(mmk *)
---

# Recipe: Bulk AI Summary for Notion Database

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Uses: [`mmk-notion-database`](../mmk-notion-database/SKILL.md) — Schema, query, and AI summary commands.

A multi-step workflow that discovers AI configuration for a database, collects all page IDs, and triggers AI summary generation in batches.

**This recipe has side effects** (triggers AI summary writes on Notion pages). It requires manual invocation only.

## Workflow

### Step 1: Discover database properties

```bash
mmk notion database schema --database-id <db-uuid> -o json
```

Review the schema output to understand the database structure and confirm it has an AI summary property.

### Step 2: Detect AI configuration

```bash
mmk notion database schema --database-id <db-uuid> --detect-ai --page-id <page-uuid> -o json
```

Pick any existing page ID from the database. The output provides:
- `property-id` — the AI summary property identifier
- `workflow-id` — the AI workflow identifier

Both values are needed for the `ai-summary` command. Save them for Step 4.

### Step 3: Collect all page IDs

```bash
mmk notion database query --database-id <db-uuid> -o json
```

Collect all page IDs from the query results. If the database has more than 100 pages, use `--cursor` for pagination:

```bash
mmk notion database query --database-id <db-uuid> --cursor "<next-cursor>" -o json
```

Repeat until all pages are collected.

### Step 4: Trigger AI summary in batches

```bash
mmk notion database ai-summary --database-id <db-uuid> --page-id <id-1> --page-id <id-2> ... --property-id "<property-id>" --workflow-id "<workflow-id>" -o json
```

**Limits:** Max 10 page IDs per execution. Split collected pages into batches of 10 or fewer.

For each batch:
1. Run the `ai-summary` command with up to 10 `--page-id` flags
2. Report progress (e.g., "Batch 2/5 complete — 20/47 pages processed")
3. Continue to the next batch

### Step 5: Validate summaries

```bash
mmk notion database query --database-id <db-uuid> -o json
```

Query the database again and check that AI summary fields are now populated across the pages.

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `database-id` | Yes | Notion database ID or URL |
| `page-id` | Yes (Step 2) | Any existing page ID for AI config detection |

## Example Session

```
User: /mmk-notion-database-ai-summary

Claude: Which database would you like to generate AI summaries for?

User: Database ID is abc123-def456. Use page xyz789 to detect the AI config.

Claude: [Step 1] Fetching database schema...
       Found 5 properties: Name, Status, Tags, Summary, Created.

       [Step 2] Detecting AI configuration...
       Found property-id: "prop_abc" and workflow-id: "wf_123".

       [Step 3] Collecting all page IDs...
       Found 23 pages in the database.

       [Step 4] Triggering AI summaries in batches of 10...
       Batch 1/3 complete — 10/23 pages processed.
       Batch 2/3 complete — 20/23 pages processed.
       Batch 3/3 complete — 23/23 pages processed.

       [Step 5] Validating...
       All 23 pages now have AI summaries populated.
       Done.
```
