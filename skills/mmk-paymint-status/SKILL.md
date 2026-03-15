---
name: mmk-paymint-status
description: Check Paymint invoice status. Triggers on "payment status", "check invoice", "invoice status", "check payment".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Check Status

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## status — Check invoice status

```bash
mmk paymint status --tid TID123 -o json
```

**Required:** `--tid` (transaction/bill ID)
**Optional:** `--business-number`

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send payment invoice
- [mmk-paymint-cancel](../mmk-paymint-cancel/SKILL.md) — Cancel/refund invoice
