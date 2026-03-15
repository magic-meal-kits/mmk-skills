---
name: mmk-paymint-resend
description: Resend Paymint invoice SMS notification. Triggers on "resend SMS", "resend invoice", "send invoice again", "re-notify".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Resend SMS

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## resend — Resend invoice SMS

```bash
mmk paymint resend --tid TID123 -o json
```

**Required:** `--tid`
**Optional:** `--business-number`

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send payment invoice
- [mmk-paymint-status](../mmk-paymint-status/SKILL.md) — Check invoice status
