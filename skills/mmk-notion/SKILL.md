---
name: mmk-notion
description: This skill should be used when the user asks to manage Notion pages, workspaces, teams, people, databases, meetings, comments, users, publishing, subscriptions, or any Notion-related operation using the mmk CLI. Triggers on phrases like "invite to Notion", "publish page", "revoke access", "list teams", "database schema", "AI summary", "meeting notes", "workspace members", "duplicate page", "page config", "publish settings", "list published", "guest pages", "subscription", "add comment", "reply to comment", "list users", "get user", "get page", "update page", "archive page", "lock page".
allowed-tools: Bash(mmk *)
---

# MMK Notion

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

Manage Notion pages, workspaces, teams, people, databases, and meetings using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `page` | Invite, revoke, publish, unpublish, config, publish-settings, duplicate, list-published, markdown, transcript, get, update (12 commands) | [`mmk-notion-page`](../mmk-notion-page/SKILL.md) |
| `workspace` | Invite users to workspace, remove from workspace (2 commands) | [`mmk-notion-workspace`](../mmk-notion-workspace/SKILL.md) |
| `team` | List teams, invite to team, remove from team (3 commands) | [`mmk-notion-team`](../mmk-notion-team/SKILL.md) |
| `subscription` | Show workspace subscription details (1 command) | [`mmk-notion-subscription`](../mmk-notion-subscription/SKILL.md) |
| `people` | List members/guests, show guest page access (2 commands) | [`mmk-notion-people`](../mmk-notion-people/SKILL.md) |
| `database` | Schema, query, insert, update, upsert, delete, AI summary, AI summary auto (8 commands) | [`mmk-notion-database`](../mmk-notion-database/SKILL.md) |
| `comment` | Add, list, reply page comments (3 commands) | [`mmk-notion-comment`](../mmk-notion-comment/SKILL.md) |
| `user` | Get user details, list workspace users (2 commands) | [`mmk-notion-user`](../mmk-notion-user/SKILL.md) |
| `meeting` | List AI meeting notes (1 command) | [`mmk-notion-meeting`](../mmk-notion-meeting/SKILL.md) |

## Tips

1. `--block-id` accepts both raw UUIDs and full Notion URLs — the CLI extracts the ID automatically
2. For bulk operations, run commands sequentially and check each result
3. Always use `-o json` for reliable parsing in scripts and skill compositions
4. Page invite errors for non-Notion users indicate the email is not associated with a Notion account — consider using the [`mmk-notion-onboard`](../mmk-notion-onboard/SKILL.md) skill for automatic email fallback
