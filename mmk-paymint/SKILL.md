---
name: mmk-paymint
description: This skill should be used when the user asks to send invoices, check payment status, cancel/refund payments, resend invoice SMS, list licenses, or any Paymint billing operation using the mmk CLI. Triggers on phrases like "send invoice", "payment status", "cancel payment", "refund", "resend SMS", "list licenses", "billing", "paymint".
allowed-tools: Bash(mmk *)
---

# MMK Paymint

Manage payment invoices using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

## License Resolution

All commands except `licenses` resolve a Paymint license automatically:
- **Single license**: `--business-number` is optional (auto-selected)
- **Multiple licenses**: `--business-number` is required to select which license to use
- Accepts: full business number, member ID, or masked number match (e.g., `305-**-***47` matches `305-95-87247`)

Run `mmk paymint licenses` first to see available licenses.

---

## Commands

### licenses — List approved Paymint licenses

```bash
mmk paymint licenses -o json
```

No flags required.

### send — Send payment invoice

```bash
mmk paymint send --phone 01012345678 --amount 10000 --product "Monthly Fee" --name "John" --message "Payment due" --expire 2026-04-01 -o json
```

**Required:** `--phone`, `--amount`, `--product`, `--name`, `--message`, `--expire` (YYYY-MM-DD)
**Optional:** `--business-number` (required if multiple licenses exist)

### status — Check invoice status

```bash
mmk paymint status --tid TID123 -o json
```

**Required:** `--tid` (transaction/bill ID)
**Optional:** `--business-number`

### cancel — Cancel/refund invoice

```bash
mmk paymint cancel --tid TID123 --amount 10000 -o json
```

**Required:** `--tid`, `--amount` (refund amount)
**Optional:** `--business-number`

### resend — Resend invoice SMS

```bash
mmk paymint resend --tid TID123 -o json
```

**Required:** `--tid`
**Optional:** `--business-number`

---

## Error Patterns

| Error | Meaning | Action |
|-------|---------|--------|
| `"no approved paymint licenses found"` | No active license | Purchase at make.magicmealkits.com |
| `"multiple licenses available, specify --business-number"` | Ambiguous license | Provide `--business-number` |
| `"no license found for ..."` | Business number not matched | Check with `mmk paymint licenses` |
