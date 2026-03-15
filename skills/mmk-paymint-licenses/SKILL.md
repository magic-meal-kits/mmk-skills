---
name: mmk-paymint-licenses
description: List approved Paymint licenses. Triggers on "list licenses", "paymint licenses", "available licenses", "business number".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Licenses

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## licenses — List approved Paymint licenses

```bash
mmk paymint licenses -o json
```

No flags required.

Run this first to see available licenses before using other Paymint commands.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send payment invoice
