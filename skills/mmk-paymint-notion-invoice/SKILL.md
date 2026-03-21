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

## Database Schema Requirements

The recipe discovers properties by their **Notion type and semantic meaning**, not by exact name. English and Korean property names are both supported automatically.

### Required Properties

| Purpose | Notion Type | Maps to | Auto-match names |
|---------|-------------|---------|------------------|
| Recipient name | title or rich_text | `--name` | (title property, or rich_text: 고객명, 이름, 성명, Name) |
| Phone number | phone_number or rich_text | `--phone` | Phone, 연락처, 전화번호, 휴대폰, 핸드폰 |
| Invoice amount | number | `--amount` | Amount, 결제금액, 금액, 청구금액 |
| Product description | rich_text | `--product` | Product, 상품명, 상품, 제품, 서비스 |
| Invoice message | rich_text | `--message` | Message, 메시지, 내용, 안내문구 |
| Invoice status | status or select | Filter key | Status, 결제상태, 상태, 진행상태 |

### Optional Properties (auto-detected for write-back)

| Purpose | Notion Type | Auto-match names |
|---------|-------------|------------------|
| Expiry date | date | Expire, 만료일, 만료 |
| Transaction/bill ID | rich_text | TID, 청구서ID, bill_id |
| Error message | rich_text | Error, 오류, 에러, 발송결과메시지 |
| Sent timestamp | date | SentAt, 발송일시, 발송일 |
| Payment link | url | ShortURL, 결제링크, 링크 |
| Result code | rich_text | 발송결과코드, ResultCode |

### Notes

- **Title property limitation:** The title property has update restrictions in the Notion API. Do not write bill_id or other result data into the title field. Use a dedicated rich_text field instead.
- **Type-first matching:** The title type is unique per database (always unambiguous). The phone_number type is also typically unambiguous. For number and rich_text types, name patterns resolve ambiguity.
- **Ambiguity handling:** If multiple properties match a required purpose, present options and ask the user to choose.

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

#### 1a. Property discovery

Map database properties to required purposes using **type-first, then name-pattern** matching:

1. **Name:** Find `rich_text` matching name patterns: `고객명`, `이름`, `성명`, `Name`. If none found, fall back to the single `title` type property.
2. **Phone:** Find `phone_number` type property. If none, find `rich_text` matching name patterns: `Phone`, `연락처`, `전화번호`, `휴대폰`, `핸드폰`.
3. **Amount:** Find `number` type properties. If exactly one, use it. If multiple, match by name: `Amount`, `결제금액`, `금액`, `청구금액`. If still ambiguous, ask the user.
4. **Status:** Find `status` or `select` type properties matching name patterns: `Status`, `결제상태`, `상태`, `진행상태`. If multiple candidates, ask the user.
5. **Product:** Find `rich_text` matching: `Product`, `상품명`, `상품`, `제품`, `서비스`.
6. **Message:** Find `rich_text` matching: `Message`, `메시지`, `내용`, `안내문구`.

If any required purpose has zero candidates, stop with a clear error listing what is missing and what types/names are expected.

Present the complete mapping:

```
Property mapping (auto-detected):
  Name    -> "고객명" (title)
  Phone   -> "연락처" (phone_number)
  Amount  -> "결제금액" (number)
  Product -> "상품명" (rich_text)
  Message -> "안내문구" (rich_text)
  Status  -> "결제상태" (status)
```

#### 1b. Status option mapping

Inspect the Status field's select/status options from the `--enhanced` schema output. Map three logical states to actual option names:

| Logical state | Used in | Default |
|---------------|---------|---------|
| **pending** | Step 2 query filter | `Pending` |
| **success** | Step 7 write-back (sent OK) | `Invoiced` |
| **failure** | Step 7 write-back (send error) | `Failed` |

**Mapping logic:**

1. Extract the Status property's option list from the enhanced schema JSON.
2. If options exist, try auto-mapping (case-insensitive):
   - **pending**: match `Pending`, `pending`, `대기`, `미발송`, `미결제`, `W-미결제`, `미발행`, `미청구`, `발송전`
   - **success**: match `Invoiced`, `Sent`, `invoiced`, `sent`, `발송완료`, `청구됨`, `완료`, `발송됨`, `청구완료`
   - **failure**: match `Failed`, `Error`, `failed`, `error`, `실패`, `오류`, `발송실패`
3. If all 3 states map confidently, show the mapping and proceed:
   ```
   Status mapping (auto-detected):
     pending  -> "미결제"
     success  -> "청구완료"
     failure  -> "발송실패"
   ```
4. If any state is ambiguous (multiple candidates or no match), present the available options and ask the user to pick which option maps to each unmapped state.
5. If the Status field has no options (e.g., plain text property), fall back to defaults: `Pending`, `Invoiced`, `Failed`.

#### 1c. Write-back field discovery

Scan remaining properties (not already mapped to a required purpose) for write-back candidates using type + name patterns:

1. **Expire:** `date` type matching `Expire`, `만료일`, `만료`
2. **TID:** `rich_text` type matching `TID`, `청구서ID`, `bill_id`
3. **Error:** `rich_text` type matching `Error`, `오류`, `에러`, `발송결과메시지`
4. **SentAt:** `date` type matching `SentAt`, `발송일시`, `발송일`
5. **ShortURL:** `url` type matching `ShortURL`, `결제링크`, `링크`
6. **ResultCode:** `rich_text` type matching `발송결과코드`, `ResultCode`

If a write-back candidate is the title property, skip it and warn: "Title property cannot be used for write-back. Use a rich_text property instead."

Present discovered write-back fields:

```
Write-back fields discovered:
  TID       -> "청구서ID" (rich_text)
  ShortURL  -> "결제링크" (url)
  SentAt    -> "발송일시" (date)
  Error     -> "발송결과메시지" (rich_text)
```

**Separate result tracking:** If dedicated result fields are found (e.g., `발송결과코드`, `발송결과메시지`), ask the user:
- (A) Update Status field after send (default)
- (B) Leave Status unchanged, only write to result fields

If no write-back fields are found at all, ask the user whether to skip write-back entirely or create the fields manually first.

### Step 2: Query pending invoices

Use the **pending** value from Step 1b and the **actual property names** discovered in Step 1a:

```bash
mmk notion database query --database-id <db-uuid> --filter "<status-property>=<mapped-pending-value>" --properties "<name>,<phone>,<amount>,<product>,<message>,<expire>" -o json
```

Replace `<status-property>`, `<name>`, `<phone>`, etc. with the actual property names discovered in Step 1a (e.g., `결제상태`, `고객명`, `연락처`).

If more than 100 records, paginate with `--cursor`:

```bash
mmk notion database query --database-id <db-uuid> --filter "<status-property>=<mapped-pending-value>" --properties "<name>,<phone>,<amount>,<product>,<message>,<expire>" --cursor "<next-cursor>" -o json
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

### Step 5: Send invoices

#### Primary path: bulk-send

Build a JSON array from all valid records and send in one call:

```bash
mmk paymint bulk-send --business-number <bn> --expire <default-expire> --data '[
  {"phone":"01012345678","amount":50000,"product":"Workshop","name":"Kim","message":"Invoice for workshop","expire":"2026-03-24"},
  {"phone":"01098765432","amount":30000,"product":"Lecture","name":"Lee","message":"Invoice for lecture"}
]' -o json
```

**Required:** `--data` (JSON array of records). Each record must have `phone`, `amount`, `product`, `name`, `message`.
**Optional:** Per-record `expire` overrides the shared `--expire` flag. `--business-number` required if multiple licenses. `--expire` defaults to 7 days from send date.

After receiving the response, check if the JSON contains recognizable field names (`bill_id`, `code`, `shortURL`, `msg`). If the response contains obfuscated/unrecognizable field names (e.g., random strings like `Av9GVZtcal`), the bulk-send response cannot be reliably parsed — fall back to sequential single send.

#### Fallback: sequential single send

If bulk-send response parsing fails, inform the user and send invoices one at a time:

```
"Bulk-send response could not be parsed (obfuscated field names).
Falling back to sending invoices one at a time."
```

```bash
mmk paymint send --phone <phone> --amount <amount> --product "<product>" --name "<name>" --message "<message>" --expire <expire> --business-number <bn> -o json
```

The single `send` response has stable, documented fields:

```json
{"bill_id":"...","code":"0000","shortURL":"https://...","msg":"성공하였습니다."}
```

| Field | Description |
|-------|-------------|
| `bill_id` | Transaction/bill identifier (use for TID write-back) |
| `code` | Result code (`"0000"` = success) |
| `shortURL` | Payment link URL |
| `msg` | Human-readable result message |

Use `code` to determine success/failure. Use `bill_id` and `shortURL` for write-back.

### Step 6: Report results

Display a final summary:

```
# Invoice Send Results
- Sent: N invoices
- Failed: M invoices
- Skipped: K records (validation errors)

## Sent Successfully
| Name | Phone | Amount | TID | ShortURL |
|------|-------|--------|-----|----------|

## Failed
| Name | Phone | Amount | Error |
|------|-------|--------|-------|

## Skipped (from Step 3)
| Name | Phone (raw) | Reason |
|------|-------------|--------|
```

### Step 7: Update Notion records

Build update payloads using the **write-back field mapping from Step 1c** and the **status mapping from Step 1b**.

For each record from Step 5/6:

**Sent successfully:**
- If Status write-back is enabled: set `<status-field>=<mapped-success-value>`
- If TID field was discovered: set `<tid-field>=<bill_id>`
- If SentAt field was discovered: set `<sentat-field>=<current datetime>`
- If ShortURL field was discovered: set `<url-field>=<shortURL>`
- If ResultCode field was discovered: set `<resultcode-field>=<code>`

**Send failed:**
- If Status write-back is enabled: set `<status-field>=<mapped-failure-value>`
- If Error field was discovered: set `<error-field>=<error message>`
- If ResultCode field was discovered: set `<resultcode-field>=<code>`

```bash
mmk notion database update --data '[
  {"page_id":"<page-id>","<status-field>":"<mapped-success-value>","<tid-field>":"bill_abc","<sentat-field>":"2026-03-22","<url-field>":"https://..."},
  {"page_id":"<page-id>","<status-field>":"<mapped-failure-value>","<error-field>":"amount below minimum"}
]' -o json
```

If any write-back target is the title property, skip writing to it and warn: "Skipping write to title property — Notion API restricts title updates. Store this value in a dedicated rich_text property instead."

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
| Required property not discovered | Stop, list missing purpose and expected types/names |
| Property discovery ambiguous | Present candidates, ask user to choose |
| No pending records found | Report "No pending invoices" and stop |
| All phones invalid | Report skipped list and stop (nothing to send) |
| Bulk-send response has obfuscated fields | Fall back to sequential single send |
| Individual send failure | Log error, continue to next record |
| Title property write-back attempted | Skip write, warn user to use rich_text field |
| Write-back partial failure | Log failed page IDs, report partial results |
| No write-back fields discovered | Ask user whether to skip or create fields manually |
| Pagination needed (>100 records) | Auto-paginate with `--cursor` |

## Example Session

```
User: /mmk-paymint-notion-invoice

Claude: Which Notion database contains your invoice records?

User: Database ID is abc123-def456

Claude: [Step 0] Resolving Paymint license...
       Found 1 license: BN 123-45-67890. Auto-selected.

       [Step 1] Discovering database schema...

       Property mapping (auto-detected):
         Name    -> "고객명" (title)
         Phone   -> "연락처" (phone_number)
         Amount  -> "결제금액" (number)
         Product -> "상품명" (rich_text)
         Message -> "안내문구" (rich_text)
         Status  -> "결제상태" (status)
         Expire  -> "만료일" (date)

       Status mapping (auto-detected from "결제상태" options):
         pending  -> "미결제"
         success  -> "청구완료"
         failure  -> "발송실패"

       Write-back fields discovered:
         TID       -> "청구서ID" (rich_text)
         ShortURL  -> "결제링크" (url)
         SentAt    -> "발송일시" (date)
         Error     -> "발송결과메시지" (rich_text)
         ResultCode -> "발송결과코드" (rich_text)

       Dedicated result fields found (발송결과코드, 발송결과메시지).
       (A) Update "결제상태" status after send (default)
       (B) Leave "결제상태" unchanged, only write to result fields
       Which option?

User: B

Claude: Status will not be updated. Results will be written to result fields only.

       [Step 2] Querying invoices (결제상태=미결제)...
       Found 5 records.

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
       Bulk-send response could not be parsed (obfuscated field names).
       Falling back to sending invoices one at a time.

       Sending 1/4: Kim... OK (bill_id: bill_abc1, shortURL: https://pay.link/abc1)
       Sending 2/4: Lee... OK (bill_id: bill_abc2, shortURL: https://pay.link/abc2)
       Sending 3/4: Choi... FAILED (amount below minimum)
       Sending 4/4: Park... OK (bill_id: bill_abc3, shortURL: https://pay.link/abc3)

       [Step 6] Results:
       Sent: 3 invoices
       Failed: 1 invoice
       Skipped: 1 record

       [Step 7] Ready to update Notion records (Status unchanged per option B):
       | Name | 청구서ID  | 결제링크                  | 발송일시   | 발송결과코드 | 발송결과메시지       |
       |------|-----------|---------------------------|------------|-------------|---------------------|
       | Kim  | bill_abc1 | https://pay.link/abc1     | 2026-03-22 | 0000        | 성공하였습니다.      |
       | Lee  | bill_abc2 | https://pay.link/abc2     | 2026-03-22 | 0000        | 성공하였습니다.      |
       | Choi |           |                           |            | 1001        | amount below minimum |
       | Park | bill_abc3 | https://pay.link/abc3     | 2026-03-22 | 0000        | 성공하였습니다.      |

       Write these updates back to Notion? (yes/no)

User: yes

Claude: [Step 7] Updating Notion records...
       4 records updated successfully.
```

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint-notion-schema](../mmk-paymint-notion-schema/SKILL.md) — Recommended Notion DB schema for Paymint invoices
- [mmk-notion-database](../mmk-notion-database/SKILL.md) — Schema and query commands
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands
- [mmk-paymint-send](../mmk-paymint-send/SKILL.md) — Send invoice command
- [mmk-paymint-licenses](../mmk-paymint-licenses/SKILL.md) — List licenses
- [mmk-paymint-status](../mmk-paymint-status/SKILL.md) — Check invoice status (for post-send verification)
