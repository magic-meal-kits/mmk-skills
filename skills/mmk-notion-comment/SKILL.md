---
name: mmk-notion-comment
description: Manage Notion page comments — add, list, reply to discussion threads. Triggers on "add comment", "list comments", "reply to comment", "page comment", "discussion reply", "comment on page".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Comment Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## add — Add a comment to a page

```bash
mmk notion comment add --page-id <page-id> --text "<comment text>" -o json
mmk notion comment add --page-id <page-id> --mention-user <user-id> --text "Please review" -o json
mmk notion comment add --page-id <page-id> --mention-page <page-id> --text "See this page" -o json
```

**Required:** `--page-id`, `--text`
**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--mention-user` | — | Mention user(s) by ID (repeatable) |
| `--mention-page` | — | Mention page(s) by ID (repeatable) |
| `--mention-database` | — | Mention database(s) by ID (repeatable) |

---

## list — List comments on a page

```bash
mmk notion comment list --page-id <page-id> -o json
mmk notion comment list --page-id <page-id> --limit 50 -o json
```

**Required:** `--page-id`
**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | `100` | Maximum number of comments to return |
| `--cursor` | — | Pagination cursor for next page of results |

---

## reply — Reply to an existing discussion thread

```bash
mmk notion comment reply --discussion-id <discussion-id> --text "<reply text>" -o json
mmk notion comment reply --discussion-id <discussion-id> --mention-user <user-id> --text "Done" -o json
```

**Required:** `--discussion-id`, `--text`
**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--mention-user` | — | Mention user(s) by ID (repeatable) |
| `--mention-page` | — | Mention page(s) by ID (repeatable) |
| `--mention-database` | — | Mention database(s) by ID (repeatable) |

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-page](../mmk-notion-page/SKILL.md) — Page invite, publish, markdown, transcript
