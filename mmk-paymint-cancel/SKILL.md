---
name: mmk-paymint-cancel
description: Cancel or refund a Paymint invoice. Triggers on "cancel payment", "refund", "cancel invoice", "refund invoice".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Cancel/Refund

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## cancel — Cancel/refund invoice

```bash
mmk paymint cancel --tid TID123 --amount 10000 -o json
```

**Required:** `--tid`, `--amount` (refund amount)
**Optional:** `--business-number`

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-status](../mmk-paymint-status/SKILL.md) — Check invoice status
- [mmk-paymint-resend](../mmk-paymint-resend/SKILL.md) — Resend invoice SMS
