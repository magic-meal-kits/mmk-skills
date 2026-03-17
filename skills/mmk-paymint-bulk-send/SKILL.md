---
name: mmk-paymint-bulk-send
description: Send multiple payment invoices at once via Paymint. Triggers on "bulk send", "batch invoice", "send multiple invoices", "bulk invoice".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Bulk Send

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## bulk-send — Send multiple payment invoices at once

```bash
mmk paymint bulk-send --data '[
  {"phone":"01012345678","amount":10000,"product":"Monthly","name":"Kim","message":"Due"},
  {"phone":"01087654321","amount":20000,"product":"Annual","name":"Lee","message":"Due"}
]' -o json

mmk paymint bulk-send --expire 2026-04-01 --callback-url https://example.com/hook --file invoices.json -o json

mmk paymint bulk-send --business-number 1234567890 --file invoices.json -o json

cat invoices.json | mmk paymint bulk-send -o json
```

**Required:** one of `--data` (inline JSON array), `--file` (path to JSON file), or piped stdin

Each record must include: `phone`, `amount`, `product`, `name`, `message`.

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--business-number` | | Business registration number or member ID (required if multiple licenses) |
| `--expire` | 7 days from today | Shared expiration date YYYY-MM-DD for all records |
| `--callback-url` | | Shared webhook callback URL for all invoices |

Per-record `expire` in the JSON overrides the shared `--expire` flag.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send single invoice
- [mmk-paymint-bulk-resend](../mmk-paymint-bulk-resend/SKILL.md) — Bulk resend SMS
- [mmk-paymint-notion-invoice](../mmk-paymint-notion-invoice/SKILL.md) — Batch invoice from Notion database (recipe)
