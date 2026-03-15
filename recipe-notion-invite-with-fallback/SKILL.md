---
name: recipe-notion-invite-with-fallback
description: This skill should be used when the user asks to "invite someone to Notion with email fallback", "invite and send signup email if not a Notion user", "onboard a user to a Notion page", or wants a workflow that tries Notion invite first and falls back to sending a Gmail signup invitation if the user doesn't have a Notion account.
disable-model-invocation: true
allowed-tools: Bash(mmk *), Bash(gws *)
---

# Recipe: Notion Invite with Gmail Fallback

A multi-step workflow that invites a user to a Notion page, and if they don't have a Notion account, sends them a signup email via Gmail.

**This recipe has side effects** (sends real emails). It requires manual invocation only.

## Prerequisites

Both CLIs must be installed and authenticated:
- `mmk` — Magic Meal Kits CLI (`npm install -g @magic-meal-kits/cli`)
- `gws` — Google Workspace CLI

## Workflow

### Step 1: Attempt Notion Page Invite

```bash
mmk notion page invite --email <email> --role <role> --block-id <block-id> -o json 2>&1
```

Check the exit code:
- **Exit 0** → Success. Report to user and stop.
- **Non-zero** → Parse the error output. Continue to Step 2.

### Step 2: Analyze the Error

Capture both stdout and stderr from the invite command:

```bash
result=$(mmk notion page invite --email <email> --role <role> --block-id <block-id> -o json 2>&1)
exit_code=$?
```

Determine the failure reason:
- If the error indicates the email is **not associated with a Notion account** (non-Notion user) → proceed to Step 3
- If it's a **different error** (auth failure, invalid block-id, permission denied) → report the error to the user and stop

### Step 3: Send Signup Email via Gmail

Send an email asking the user to create a Notion account:

```bash
gws gmail +send \
  --to "<email>" \
  --subject "Notion 페이지 초대 - 계정 생성이 필요합니다" \
  --body "안녕하세요,

Notion 페이지에 초대하려 했으나, 해당 이메일로 등록된 Notion 계정이 없습니다.

아래 링크에서 Notion 계정을 생성해 주세요:
https://www.notion.so/signup

계정 생성 후 알려주시면, 다시 페이지 접근 권한을 드리겠습니다.

감사합니다."
```

Adapt the email language and content based on the user's preferences. The above is a Korean-language template — switch to English or other languages as appropriate.

### Step 4: Confirm and Offer Retry

Report to the user:
1. The Notion invite failed because the email is not a Notion user
2. A signup email has been sent
3. Ask: "Should I retry the invite after they sign up?"

If the user confirms retry later, go back to Step 1.

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `email` | Yes | Email address of the person to invite |
| `role` | Yes | Notion permission level: `editor`, `read_and_write`, `content_only_editor`, `comment_only`, `reader` |
| `block-id` | Yes | Notion page ID or URL to invite them to |

## Example Session

```
User: /recipe-notion-invite-with-fallback

Claude: Who would you like to invite?

User: Invite kim@example.com as editor to https://notion.so/my-page-abc123

Claude: [Runs mmk notion page invite...]
       The invite failed — kim@example.com doesn't have a Notion account.
       I'll send them a signup email via Gmail.
       [Runs gws gmail +send...]
       Done. Signup email sent to kim@example.com.
       Want me to retry the invite after they create their account?
```
