---
name: mmk-paymint-bulk-resend
description: Resend SMS for multiple Paymint invoices at once. Triggers on "bulk resend", "batch resend SMS", "resend multiple invoices", "bulk re-notify".
allowed-tools: Bash(mmk *)
---

# MMK Paymint — Bulk Resend SMS

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-paymint`](../mmk-paymint/SKILL.md) — includes License Resolution and Error Patterns.

Always use `-o json` when parsing results or composing with other commands.

---

## bulk-resend — Resend SMS for multiple invoices at once

```bash
mmk paymint bulk-resend --tid TID1 --tid TID2 --tid TID3 -o json

mmk paymint bulk-resend --data '["TID1","TID2","TID3"]' -o json

mmk paymint bulk-resend --file tids.json -o json

echo '["TID1","TID2"]' | mmk paymint bulk-resend -o json
```

**Required:** one of `--tid` (repeatable), `--data` (inline JSON array of TID strings), `--file` (path to JSON file), or piped stdin

**Optional:**

| Flag | Description |
|------|-------------|
| `--business-number` | Business registration number or member ID (required if multiple licenses) |

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands overview
- [mmk-paymint-resend](../mmk-paymint-resend/SKILL.md) — Resend single invoice SMS
- [mmk-paymint-bulk-send](../mmk-paymint-bulk-send/SKILL.md) — Bulk send invoices
