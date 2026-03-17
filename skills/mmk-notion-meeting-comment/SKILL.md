---
name: mmk-notion-meeting-comment
description: Post a structured summary comment on a Notion meeting page — extracts action items and decisions from transcript. Triggers on "summarize meeting as comment", "post meeting summary", "comment meeting notes", "meeting action items comment".
disable-model-invocation: true
allowed-tools: Bash(mmk *)
---

# Recipe: Meeting Notes Commentator

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Uses: [`mmk-notion-page`](../mmk-notion-page/SKILL.md) — Page transcript command.
> Uses: [`mmk-notion-comment`](../mmk-notion-comment/SKILL.md) — Comment add command.

A multi-step workflow that extracts a meeting page transcript, summarizes action items and key decisions, and posts a structured comment back to the page.

**This recipe has side effects** (posts comments on Notion pages). It requires manual invocation only.

## Workflow

### Step 1: Get meeting page transcript

```bash
mmk notion page transcript <page_id> --include summary,notes -o json
```

Retrieve the meeting transcript with summary and notes sections.

### Step 2: Extract action items and decisions

Analyze the transcript output and extract:
- Action items (who, what, when)
- Key decisions made
- Follow-up items

### Step 3: Format structured comment

Format the extracted information as a structured comment:

```
Action Items:
- [ ] @person: Task description (by date)

Key Decisions:
- Decision description

Follow-ups:
- Follow-up item
```

Adapt the format and language based on the transcript content and user preferences.

### Step 4: Post comment to the meeting page

```bash
mmk notion comment add --page-id <page_id> --text "<formatted summary>" -o json
```

### Step 5: Confirm to user

Report:
1. Number of action items, decisions, and follow-ups extracted
2. Comment posted successfully
3. Offer to adjust or add more detail

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `page_id` | Yes | Notion meeting page ID or URL |

## Example Session

```
User: /mmk-notion-meeting-comment

Claude: Which meeting page would you like me to summarize?

User: https://notion.so/meeting-abc123

Claude: [Step 1] Fetching meeting transcript...
       Found transcript with summary and notes.

       [Step 2] Extracting action items and decisions...
       Found 3 action items, 2 key decisions, 1 follow-up.

       [Step 3] Formatting structured comment...

       [Step 4] Posting comment to the meeting page...
       Comment posted successfully.

       Summary posted:
       - 3 action items assigned
       - 2 key decisions documented
       - 1 follow-up noted

       Would you like me to adjust anything?
```
