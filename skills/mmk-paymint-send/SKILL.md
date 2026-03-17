---
name: mmk-paymint-send
description: Send payment invoice via Paymint. Triggers on "send invoice", "create invoice", "bill someone", "payment request".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Send Invoice

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## send — Send payment invoice

```bash
mmk paymint send --phone 01012345678 --amount 10000 --product "Monthly Fee" --name "John" --message "Payment due" --expire 2026-04-01 -o json
```

**Required:** `--phone`, `--amount`, `--product`, `--name`, `--message`, `--expire` (YYYY-MM-DD)
**Optional:** `--business-number` (required if multiple licenses exist)

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-status](../mmk-paymint-status/SKILL.md) — Check invoice status
- [mmk-paymint-cancel](../mmk-paymint-cancel/SKILL.md) — Cancel/refund invoice
- [mmk-paymint-notion-invoice](../mmk-paymint-notion-invoice/SKILL.md) — Batch invoice from Notion database (recipe)
