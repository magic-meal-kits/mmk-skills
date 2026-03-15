---
name: mmk-paymint
description: This skill should be used when the user asks to send invoices, check payment status, cancel/refund payments, resend invoice SMS, list licenses, or any Paymint billing operation using the mmk CLI. Triggers on phrases like "send invoice", "payment status", "cancel payment", "refund", "resend SMS", "list licenses", "billing", "paymint".
allowed-tools: Bash(mmk *)
---

# MMK Paymint

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

Manage payment invoices using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `licenses` | List approved Paymint licenses | [`mmk-paymint-licenses`](../mmk-paymint-licenses/SKILL.md) |
| `send` | Send payment invoice | [`mmk-paymint-send`](../mmk-paymint-send/SKILL.md) |
| `status` | Check invoice status | [`mmk-paymint-status`](../mmk-paymint-status/SKILL.md) |
| `cancel` | Cancel/refund invoice | [`mmk-paymint-cancel`](../mmk-paymint-cancel/SKILL.md) |
| `resend` | Resend invoice SMS | [`mmk-paymint-resend`](../mmk-paymint-resend/SKILL.md) |

## License Resolution

All commands except `licenses` resolve a Paymint license automatically:
- **Single license**: `--business-number` is optional (auto-selected)
- **Multiple licenses**: `--business-number` is required to select which license to use
- Accepts: full business number, member ID, or masked number match (e.g., `305-**-***47` matches `305-95-87247`)

Run `mmk paymint licenses` first to see available licenses.

## Error Patterns

| Error | Meaning | Action |
|-------|---------|--------|
| `"no approved paymint licenses found"` | No active license | Purchase at make.magicmealkits.com |
| `"multiple licenses available, specify --business-number"` | Ambiguous license | Provide `--business-number` |
| `"no license found for ..."` | Business number not matched | Check with `mmk paymint licenses` |
