---
name: mmk-notion-meeting
description: List Notion AI meeting notes. Triggers on "meeting notes", "AI meetings", "list meetings", "meeting list".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Meeting Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## list — List AI meeting notes

```bash
mmk notion meeting list -o json
mmk notion meeting list --timezone Asia/Seoul --limit 50 -o json
```

**Optional:** `--view-id`, `--timezone` (default: `Asia/Tokyo`), `--limit` (default: 25)

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
