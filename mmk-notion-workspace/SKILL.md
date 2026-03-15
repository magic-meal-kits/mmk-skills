---
name: mmk-notion-workspace
description: Manage Notion workspace membership — invite users to workspace, remove users from workspace. Triggers on "invite to workspace", "add to workspace", "remove from workspace", "workspace member".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Workspace Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## invite — Invite a user to the workspace

```bash
mmk notion workspace invite --email <email> --role <role> -o json
```

**Required:** `--email`, `--role` (`owner` or `member`)
**Optional:** `--message`

---

## remove — Remove a user from the workspace

```bash
mmk notion workspace remove --email <email> -o json
```

**Required:** `--email`

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-page](../mmk-notion-page/SKILL.md) — Page-level invite and permissions
- [mmk-notion-people](../mmk-notion-people/SKILL.md) — List workspace members and guests
