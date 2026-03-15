---
name: mmk-notion-page
description: Manage Notion pages — invite, revoke, publish, unpublish, config, publish-settings, duplicate, list-published. Triggers on "invite to page", "revoke access", "publish page", "unpublish", "page config", "full width", "publish settings", "duplicate page", "list published".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Page Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## invite — Invite a user to a page

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

---

## revoke — Revoke a user's access to a page

```bash
mmk notion page revoke --email <email> --block-id <page-id-or-url> -o json
```

**Required:** `--email`, `--block-id`

---

## publish — Publish a page to the web

```bash
mmk notion page publish --block-id <page-id-or-url> -o json
```

**Required:** `--block-id`

Returns public URL on success. Returns 409 if already published (not an error — returns existing public URL).

---

## unpublish — Unpublish a page from the web

```bash
mmk notion page unpublish --block-id <page-id-or-url> -o json
```

**Required:** `--block-id`

Returns 409 if already private (not an error).

---

## config — Configure page display settings

```bash
mmk notion page config --block-id <page-id> --full-width -o json
mmk notion page config --block-id id1 --block-id id2 --small-text=false -o json
```

**Required:** `--block-id` (string slice — supports multiple pages)
**At least one of:** `--full-width` (bool), `--small-text` (bool)

---

## publish-settings — Update publish settings for a published page

```bash
mmk notion page publish-settings --block-id <id> --allow-duplicate=false -o json
mmk notion page publish-settings --block-id <id> --seo-title "Title" --slug "my-page" -o json
```

**Required:** `--block-id`
**At least one of:** `--allow-duplicate` (bool), `--search-engine-index` (bool), `--seo-title`, `--seo-description`, `--social-image`, `--slug`

---

## duplicate — Duplicate a page into a parent page

```bash
mmk notion page duplicate --source-id <page-id> --parent-id <parent-id> -o json
```

**Required:** `--source-id`, `--parent-id`

---

## list-published — List all published pages in the workspace

```bash
mmk notion page list-published -o json
mmk notion page list-published --type forms -o json
```

**Optional:** `--type` — `sites` (default), `forms`, or `links`

---

## Tips

1. `--block-id` accepts both raw UUIDs and full Notion URLs — the CLI extracts the ID automatically
2. For bulk operations, run commands sequentially and check each result
3. Page invite errors for non-Notion users indicate the email is not associated with a Notion account — consider using the [`mmk-notion-onboard`](../mmk-notion-onboard/SKILL.md) skill for automatic email fallback

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-workspace](../mmk-notion-workspace/SKILL.md) — Workspace invite and remove
- [mmk-notion-people](../mmk-notion-people/SKILL.md) — List members, guest page access
