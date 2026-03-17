---
name: mmk-paymint-notion-invoice
description: Batch send Paymint invoices from a Notion database with Notion write-back. Triggers on "batch invoice from notion", "send invoices from database", "notion invoice", "paymint notion", "bulk invoice send".
disable-model-invocation: true
allowed-tools: Bash(mmk *)
---

# Recipe: Batch Paymint Invoice from Notion Database

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Uses: [`mmk-notion-database`](../mmk-notion-database/SKILL.md) — Schema and query commands.
> Uses: [`mmk-paymint-send`](../mmk-paymint-send/SKILL.md) — Send invoice command.
> Uses: [`mmk-paymint-licenses`](../mmk-paymint-licenses/SKILL.md) — License resolution.

A multi-step workflow that reads invoice records from a Notion database, validates phone numbers, previews with dry-run, sends Paymint invoices in bulk, and writes results back to Notion.

**This recipe has side effects** (sends real invoices with payment links via SMS). It requires manual invocation only.

## Recommended Notion Database Schema

| Property | Notion Type | Maps to | Notes |
|----------|-------------|---------|-------|
| `Name` | Title | `--name` | Recipient name |
| `Phone` | Phone or Rich Text | `--phone` | Korean mobile number |
| `Amount` | Number | `--amount` | Invoice amount in KRW |
| `Product` | Rich Text | `--product` | Product/service description |
| `Message` | Rich Text | `--message` | Invoice message |
| `Expire` | Date | `--expire` | Per-row expiry (YYYY-MM-DD). Default: 7 days from send date if empty |
| `Status` | Status or Select | Filter key | `Pending` / `Invoiced` / `Failed` / `Paid` / `Cancelled` |
| `TID` | Rich Text | — | Transaction ID after send (for status/cancel/resend) |
| `Error` | Rich Text | — | Error message on failure |
| `SentAt` | Date | — | When invoice was sent |

## Phone Number Normalization Rules

Apply these rules inline per record before sending:

1. Strip all non-digit characters (`-`, spaces, `+`)
2. If starts with `82` and remaining 10 digits start with `10` — replace `82` with `0` (international format)
3. If 10 digits starting with `10` — prepend `0` (`1012345678` -> `01012345678`)
4. If 11 digits matching `^010[0-9]{8}$` — valid
5. Otherwise — invalid, skip with reason

**Examples:**

| Input | Result | Reason |
|-------|--------|--------|
| `010-1234-5678` | `01012345678` | Strip hyphens |
| `1012345678` | `01012345678` | Prepend `0` (10-digit shorthand) |
| `+82-10-1234-5678` | `01012345678` | International to domestic |
| `8201012345678` | `01012345678` | International with leading 82 |
| `021234567` | INVALID | Landline number |
| `0101234` | INVALID | Too short |

## Workflow

### Step 0: Resolve Paymint license

```bash
mmk paymint licenses -o json
```

If 1 license exists, auto-select it. If multiple, present the list and ask the user which `--business-number` to use for all sends.

### Step 1: Discover and validate database schema

```bash
mmk notion database schema --database-id <db-uuid> --enhanced -o json
```

Confirm that the required properties exist: `Name`, `Phone`, `Amount`, `Product`, `Message`, `Status`. Stop with a clear error if any critical property is missing.

### Step 2: Query pending invoices

```bash
mmk notion database query --database-id <db-uuid> --filter "Status=Pending" --properties "Name,Phone,Amount,Product,Message,Expire" -o json
```

If more than 100 records, paginate with `--cursor`:

```bash
mmk notion database query --database-id <db-uuid> --filter "Status=Pending" --properties "Name,Phone,Amount,Product,Message,Expire" --cursor "<next-cursor>" -o json
```

Repeat until all pending records are collected.

### Step 3: Validate and normalize phone numbers + defaults

For each record:
1. Apply phone normalization rules above
2. Separate into **valid** and **invalid** lists
3. For rows with empty `Expire`: default to **today + 7 days** (YYYY-MM-DD)

### Step 4: Dry-run preview

Display a summary table of valid records:

```
# Valid Records (ready to send)
| # | Name | Phone | Amount | Product | Expire | Notes |
|---|------|-------|--------|---------|--------|-------|
| 1 | Kim  | 01012345678 | 50000 | Workshop | 2026-03-24 | Expire defaulted |
| 2 | Lee  | 01098765432 | 30000 | Lecture  | 2026-04-01 | |

# Skipped Records (invalid)
| # | Name | Phone (raw) | Reason |
|---|------|-------------|--------|
| 1 | Park | 021234567   | Invalid: landline number |
```

**Wait for explicit user confirmation before proceeding.** If the user declines, stop.

### Step 5: Bulk-send invoices

Build a JSON array from all valid records and send in one call:

```bash
mmk paymint bulk-send --business-number <bn> --expire <default-expire> --data '[
  {"phone":"01012345678","amount":50000,"product":"Workshop","name":"Kim","message":"Invoice for workshop","expire":"2026-03-24"},
  {"phone":"01098765432","amount":30000,"product":"Lecture","name":"Lee","message":"Invoice for lecture"}
]' -o json
```

**Required:** `--data` (JSON array of records). Each record must have `phone`, `amount`, `product`, `name`, `message`.
**Optional:** Per-record `expire` overrides the shared `--expire` flag. `--business-number` required if multiple licenses. `--expire` defaults to 7 days from send date.

The response includes per-record results:
- **Success** — includes TID for the record
- **Failure** — includes error message for the record
- All records are attempted regardless of individual failures

### Step 6: Report results

Display a final summary:

```
# Invoice Send Results
- Sent: N invoices
- Failed: M invoices
- Skipped: K records (validation errors)

## Sent Successfully
| Name | Phone | Amount | TID |
|------|-------|--------|-----|

## Failed
| Name | Phone | Amount | Error |
|------|-------|--------|-------|

## Skipped (from Step 3)
| Name | Phone (raw) | Reason |
|------|-------------|--------|
```

### Step 7: Update Notion records

Build update payloads from the send results and write back to Notion:

```bash
mmk notion database update --data '[
  {"page_id":"<page-id>","Status":"Invoiced","TID":"txn_abc","SentAt":"2026-03-17"},
  {"page_id":"<page-id>","Status":"Failed","Error":"amount below minimum"}
]' -o json
```

For each record from Step 5:
- **Sent successfully** — set `Status=Invoiced`, `TID=<tid>`, `SentAt=<current datetime>`
- **Send failed** — set `Status=Failed`, `Error=<error message>`

**Wait for explicit user confirmation before writing back to Notion.** This is a separate confirmation from Step 4. Display the planned updates as a preview table before asking.

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `database-id` | Yes | Notion database ID or URL |
| `business-number` | No | Paymint license business number (required if multiple licenses) |

## Error Handling

| Scenario | Action |
|----------|--------|
| Auth failure on any command | Stop workflow, report `mmk auth login` needed |
| Database not found | Stop, ask user to verify `database-id` |
| Missing required properties in schema | Stop, list missing properties |
| No pending records found | Report "No pending invoices" and stop |
| All phones invalid | Report skipped list and stop (nothing to send) |
| Individual send failure | Log error, continue to next record |
| Pagination needed (>100 records) | Auto-paginate with `--cursor` |

## Example Session

```
User: /mmk-paymint-notion-invoice

Claude: Which Notion database contains your invoice records?

User: Database ID is abc123-def456

Claude: [Step 0] Resolving Paymint license...
       Found 1 license: BN 123-45-67890. Auto-selected.

       [Step 1] Checking database schema...
       Found required properties: Name, Phone, Amount, Product, Message, Status.
       Optional properties found: Expire, TID, Error, SentAt.

       [Step 2] Querying pending invoices...
       Found 5 records with Status=Pending.

       [Step 3] Validating phone numbers...
       4 valid, 1 invalid.

       [Step 4] Preview:

       Ready to send:
       | # | Name | Phone       | Amount | Product  | Expire     |
       |---|------|-------------|--------|----------|------------|
       | 1 | Kim  | 01012345678 | 50000  | Workshop | 2026-03-24 |
       | 2 | Lee  | 01098765432 | 30000  | Lecture  | 2026-04-01 |
       | 3 | Choi | 01055556666 | 25000  | Material | 2026-03-24 |
       | 4 | Park | 01077778888 | 40000  | Fee      | 2026-03-30 |

       Skipped:
       | Name | Phone (raw) | Reason |
       |------|-------------|--------|
       | Jung | 021234567   | Invalid: landline number |

       Proceed with sending 4 invoices? (yes/no)

User: yes

Claude: [Step 5] Bulk-sending 4 invoices...
       Results:
       - Kim: OK (TID: txn_abc1)
       - Lee: OK (TID: txn_abc2)
       - Choi: FAILED (amount below minimum)
       - Park: OK (TID: txn_abc3)

       [Step 6] Results:
       Sent: 3 invoices
       Failed: 1 invoice
       Skipped: 1 record

       [Step 7] Ready to update Notion records:
       | Name | Status   | TID       | Error                |
       |------|----------|-----------|----------------------|
       | Kim  | Invoiced | txn_abc1  |                      |
       | Lee  | Invoiced | txn_abc2  |                      |
       | Choi | Failed   |           | amount below minimum |
       | Park | Invoiced | txn_abc3  |                      |

       Write these updates back to Notion? (yes/no)

User: yes

Claude: [Step 7] Updating Notion records...
       4 records updated successfully.
```

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion-database](../mmk-notion-database/SKILL.md) — Schema and query commands
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send invoice command
- [mmk-paymint-licenses](../mmk-paymint-licenses/SKILL.md) — List licenses
- [mmk-paymint-status](../mmk-paymint-status/SKILL.md) — Check invoice status (for post-send verification)
