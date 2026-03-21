---
name: mmk-paymint-notion-schema
description: Recommended Notion database schema for Paymint invoice tracking. Triggers on "paymint notion schema", "paymint database setup", "invoice database design", "paymint notion template", "paymint db schema".
allowed-tools: Bash(mmk *)
---

# Paymint Notion Database — Recommended Schema

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Related: [`mmk-paymint-notion-invoice`](../mmk-paymint-notion-invoice/SKILL.md) — Batch send workflow that uses this schema.

This skill shows the recommended Notion database schema for Paymint invoice lifecycle tracking. Present the appropriate tier based on user needs.

## Design Tiers

### Minimal (11 properties)

Just enough for the [`mmk-paymint-notion-invoice`](../mmk-paymint-notion-invoice/SKILL.md) recipe. Use this when the user only needs to send invoices and track basic results.

| Property | Type | Category | Notes |
|----------|------|----------|-------|
| (title) | title | Input | Descriptive label (e.g., "김철수 - 3월 워크샵") |
| 고객명 | rich_text | Input | Recipient name (maps to `--name`) |
| 연락처 | phone_number | Input | Korean mobile number (maps to `--phone`) |
| 결제금액 | number | Input | Invoice amount in KRW (maps to `--amount`) |
| 청구사유 | rich_text | Input | Product/service description (maps to `--product`) |
| 안내메시지 | rich_text | Input | Invoice message (maps to `--message`) |
| 유효기간 | date | Input | Expiry date (YYYY-MM-DD). Default: 7 days from send |
| 결제상태 | select | Status | Options: `미결제`, `청구완료`, `발송실패`, `결제완료`, `취소` |
| 청구서ID | rich_text | Result | bill_id from send response |
| 발송결과코드 | rich_text | Result | Result code ("0000" = success) |
| ShortURL | url | Result | Payment link |

### Full (31 properties)

Complete invoice lifecycle tracking including payment callbacks and cancellations. Use this for production databases that need to track the full payment flow.

---

## 1. Input Fields

Properties the user fills in before sending invoices.

| Property | Type | Notes |
|----------|------|-------|
| (title) | title | Descriptive label (e.g., "김철수 - 3월 워크샵"). Row identifier in Notion list view. |
| 고객명 | rich_text | Recipient name (maps to `--name`) |
| 연락처 | phone_number | Korean mobile number (maps to `--phone`) |
| 결제금액 | number | Invoice amount in KRW (maps to `--amount`) |
| 청구사유 | rich_text | Product/service description (maps to `--product`) |
| 안내메시지 | rich_text | Invoice message (maps to `--message`) |
| 유효기간 | date | Expiry date (YYYY-MM-DD). Default: 7 days from send date if empty |
| 이메일 | email | Optional: recipient email for records |

## 2. Status & Send Result Fields

Written by the [`mmk-paymint-notion-invoice`](../mmk-paymint-notion-invoice/SKILL.md) recipe after sending.

| Property | Type | Notes |
|----------|------|-------|
| 결제상태 | select | `미결제` / `청구완료` / `발송실패` / `결제완료` / `취소` |
| 청구서ID | rich_text | bill_id from send response. Must be rich_text, NOT title (title has update restrictions). |
| 발송결과코드 | rich_text | Result code ("0000" = success) |
| 발송결과메시지 | rich_text | Human-readable result message |
| ShortURL | url | Payment link sent to customer |
| CallbackURL | url | Webhook callback URL (if configured) |

### 결제상태 Options

| Option | Meaning | Set by |
|--------|---------|--------|
| `미결제` | Pending — invoice not yet sent | Initial state |
| `청구완료` | Invoiced — send succeeded | Recipe write-back |
| `발송실패` | Failed — send error | Recipe write-back |
| `결제완료` | Paid — customer completed payment | Payment callback |
| `취소` | Cancelled/refunded | Cancel operation |

## 3. Payment Result Fields

Populated by the payment system callback when the customer pays. The recipe does NOT write these — they come from the payment gateway webhook.

| Property | Type | Notes |
|----------|------|-------|
| 결제수단 | select | Payment method (card, transfer, etc.) |
| 승인번호 | rich_text | Approval number |
| 승인금액 | number | Approved amount |
| 승인일시 | rich_text | Approval datetime |
| 카드/계좌번호 | rich_text | Masked card/account number |
| 카드/은행코드 | rich_text | Card/bank code |
| 카드/은행명 | rich_text | Card/bank name |
| 매입사코드 | rich_text | Acquirer code |
| 매입사명 | rich_text | Acquirer name |
| 할부개월 | rich_text | Installment months |

## 4. Cancel Fields

Populated on cancel/refund operations.

| Property | Type | Notes |
|----------|------|-------|
| 취소결과코드 | rich_text | Cancel result code |
| 취소결과메시지 | rich_text | Cancel result message |
| 취소일시 | rich_text | Cancel datetime |
| 원거래승인번호 | rich_text | Original approval number |

## 5. Meta Fields

| Property | Type | Notes |
|----------|------|-------|
| 메모 | rich_text | Internal notes |
| 생성일 | created_time | Auto-set by Notion |
| 수정일 | last_edited_time | Auto-set by Notion |

---

## Important Design Notes

- **Title is a descriptive label**, not a data field. Use it for human-readable row identification in Notion views (e.g., "김철수 - 3월 워크샵"). The title property has update restrictions in the Notion API, so never use it for write-back targets like 청구서ID.
- **고객명 is rich_text**, not title. This keeps the customer name as a separate queryable and writable field that the invoice recipe can reliably map.
- **청구서ID must be rich_text.** The recipe writes bill_id here after sending. If this were title type, the write-back would fail.
- **Category 3 (Payment Result) and 4 (Cancel) fields** are populated by external systems (payment gateway callbacks), not by the recipe. Include them only if your system has webhook integration.
- **결제상태 select options** cover the full lifecycle: pending → invoiced → paid (or failed/cancelled). The invoice recipe maps `미결제` as the "pending" filter and `청구완료`/`발송실패` as success/failure write-back values.

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-paymint-notion-invoice](../mmk-paymint-notion-invoice/SKILL.md) — Batch send workflow using this schema
- [mmk-notion-database](../mmk-notion-database/SKILL.md) — Schema, query, insert, update commands
- [mmk-paymint](../mmk-paymint/SKILL.md) — License Resolution, Error Patterns, all Paymint commands
