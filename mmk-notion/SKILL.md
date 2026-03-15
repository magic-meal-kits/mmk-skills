---
name: mmk-notion
description: This skill should be used when the user asks to manage Notion pages, workspaces, teams, people, databases, meetings, publishing, subscriptions, or any Notion-related operation using the mmk CLI. Triggers on phrases like "invite to Notion", "publish page", "revoke access", "list teams", "database schema", "AI summary", "meeting notes", "workspace members", "duplicate page", "page config", "publish settings", "list published", "guest pages", "subscription".
allowed-tools: Bash(mmk *)
---

# MMK Notion

Manage Notion pages, workspaces, teams, people, databases, and meetings using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

---

## Page Commands (`mmk notion page`)

### invite — Invite a user to a page

```bash
mmk notion page invite --email <email> --role <role> --block-id <page-id-or-url> -o json
```

**Required:** `--email`, `--role`, `--block-id`

**Available roles:**

| Role | Access Level | Works With |
|------|-------------|------------|
| `editor` | Full access — edit structure, content, manage permissions | Pages & Databases |
| `read_and_write` | Can edit — modify content | Pages & Databases |
| `content_only_editor` | Can edit content — edit rows but not DB structure | Databases only |
| `comment_only` | Can comment — view and add comments only | Pages & Databases |
| `reader` | Can view — read-only | Pages & Databases |

### revoke — Revoke a user's access to a page

```bash
mmk notion page revoke --email <email> --block-id <page-id-or-url> -o json
```

**Required:** `--email`, `--block-id`

### publish — Publish a page to the web

```bash
mmk notion page publish --block-id <page-id-or-url> -o json
```

**Required:** `--block-id`

Returns public URL on success. Returns 409 if already published (not an error — returns existing public URL).

### unpublish — Unpublish a page from the web

```bash
mmk notion page unpublish --block-id <page-id-or-url> -o json
```

**Required:** `--block-id`

Returns 409 if already private (not an error).

### config — Configure page display settings

```bash
mmk notion page config --block-id <page-id> --full-width -o json
mmk notion page config --block-id id1 --block-id id2 --small-text=false -o json
```

**Required:** `--block-id` (string slice — supports multiple pages)
**At least one of:** `--full-width` (bool), `--small-text` (bool)

### publish-settings — Update publish settings for a published page

```bash
mmk notion page publish-settings --block-id <id> --allow-duplicate=false -o json
mmk notion page publish-settings --block-id <id> --seo-title "Title" --slug "my-page" -o json
```

**Required:** `--block-id`
**At least one of:** `--allow-duplicate` (bool), `--search-engine-index` (bool), `--seo-title`, `--seo-description`, `--social-image`, `--slug`

### duplicate — Duplicate a page into a parent page

```bash
mmk notion page duplicate --source-id <page-id> --parent-id <parent-id> -o json
```

**Required:** `--source-id`, `--parent-id`

### list-published — List all published pages in the workspace

```bash
mmk notion page list-published -o json
mmk notion page list-published --type forms -o json
```

**Optional:** `--type` — `sites` (default), `forms`, or `links`

---

## Workspace Commands (`mmk notion workspace`)

### invite — Invite a user to the workspace

```bash
mmk notion workspace invite --email <email> --role <role> -o json
```

**Required:** `--email`, `--role` (`owner` or `member`)
**Optional:** `--message`

### remove — Remove a user from the workspace

```bash
mmk notion workspace remove --email <email> -o json
```

**Required:** `--email`

---

## Team Commands (`mmk notion team`)

### list — List teams in the workspace

```bash
mmk notion team list -o json
mmk notion team list --type All -o json
```

**Optional:** `--type` — `Joined` (default), `Pending`, or `All`

### invite — Invite a user to a team

```bash
mmk notion team invite --team-id <uuid> --email <email> -o json
```

**Required:** `--team-id`, `--email`
**Optional:** `--member-type` — `member` (default) or `owner`

### remove — Remove a user from a team

```bash
mmk notion team remove --team-id <uuid> --email <email> -o json
```

**Required:** `--team-id`, `--email`
**Optional:** `--member-type` — `member` (default) or `owner`

---

## Subscription (`mmk notion subscription`)

Show workspace subscription details (tier, type, Stripe/RevenueCat data).

```bash
mmk notion subscription -o json
```

No flags required.

---

## People Commands (`mmk notion people`)

### list — List workspace members and guests

```bash
mmk notion people list -o json
mmk notion people list --type guest -o json
```

**Optional:** `--type` — `all` (default), `member`, `guest`, or `owner`

### guest-pages — Show guest page access

```bash
mmk notion people guest-pages --user-id <uuid> -o json
mmk notion people guest-pages --user-id id1 --user-id id2 -o json
```

**Required:** `--user-id` (string slice — supports multiple user IDs)

---

## Database Commands (`mmk notion database`)

### schema — Show database property schema

```bash
mmk notion database schema --database-id <db-uuid> -o json
mmk notion database schema --database-id <db-uuid> --detect-ai --page-id <page-uuid> -o json
```

**Required:** `--database-id`
**Optional:** `--detect-ai` (bool), `--page-id` (required with `--detect-ai` — accepts UUID or full Notion URL)

When `--detect-ai` is set, output includes `property-id` and `workflow-id` values needed for the `ai-summary` command.

### ai-summary — Trigger AI summary generation for database items

```bash
mmk notion database ai-summary --database-id <db-uuid> --page-id <page-1> --page-id <page-2> --property-id "prop_id" -o json
```

**Required:** `--database-id`, `--page-id` (string slice, repeatable), `--property-id`
**Optional:** `--workflow-id`

**Limits:** Max 10 page IDs per execution. Use `schema --detect-ai` first to discover `--property-id` and `--workflow-id`.

---

## Meeting Commands (`mmk notion meeting`)

### list — List AI meeting notes

```bash
mmk notion meeting list -o json
mmk notion meeting list --timezone Asia/Seoul --limit 50 -o json
```

**Optional:** `--view-id`, `--timezone` (default: `Asia/Tokyo`), `--limit` (default: 25)

---

## Tips

1. `--block-id` accepts both raw UUIDs and full Notion URLs — the CLI extracts the ID automatically
2. For bulk operations, run commands sequentially and check each result
3. Always use `-o json` for reliable parsing in scripts and skill compositions
4. Page invite errors for non-Notion users indicate the email is not associated with a Notion account — consider using the `/mmk-notion-onboard` skill for automatic email fallback
