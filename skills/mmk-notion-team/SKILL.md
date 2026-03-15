---
name: mmk-notion-team
description: Manage Notion teams — list teams, invite users to a team, remove users from a team. Triggers on "list teams", "invite to team", "add to team", "remove from team", "team members".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Team Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## list — List teams in the workspace

```bash
mmk notion team list -o json
mmk notion team list --type All -o json
```

**Optional:** `--type` — `Joined` (default), `Pending`, or `All`

---

## invite — Invite a user to a team

```bash
mmk notion team invite --team-id <uuid> --email <email> -o json
```

**Required:** `--team-id`, `--email`
**Optional:** `--member-type` — `member` (default) or `owner`

---

## remove — Remove a user from a team

```bash
mmk notion team remove --team-id <uuid> --email <email> -o json
```

**Required:** `--team-id`, `--email`
**Optional:** `--member-type` — `member` (default) or `owner`

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-workspace](../mmk-notion-workspace/SKILL.md) — Workspace-level membership
- [mmk-notion-people](../mmk-notion-people/SKILL.md) — List members and guests
