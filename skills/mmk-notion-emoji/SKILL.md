---
name: mmk-notion-emoji
description: Manage custom Notion workspace emojis. Triggers on "custom emoji", "list emojis", "workspace emojis", "notion emoji".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Emoji Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## list — List all custom emojis in the workspace

```bash
mmk notion emoji list -o json
```

No additional flags required.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
- [mmk-notion-workspace](../mmk-notion-workspace/SKILL.md) — Workspace invite and remove
