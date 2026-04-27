<p align="center">
  <img src="assets/banner.png" alt="Magic Meal Kits — automate the impossible" width="700">
</p>

> [!WARNING]
> This project is in **beta**. Skills, command signatures, and installation methods may change without notice.

# MMK Skills

Agent skills for automating [Magic Meal Kits](https://magicmealkits.com) CLI operations. Covers Notion, Paymint, Threads, YouTube, and Plaud. Works with Claude Code, Gemini CLI, OpenCode, Codex, Cursor, and [38+ agents](https://github.com/nichochar/skills).

## Getting Started

### 1. Install the MMK CLI

```bash
npm install -g @magic-meal-kits/cli@beta
mmk config set server <your-server-url>
mmk auth login
```

### 2. Install Skills

#### Gemini CLI

Install the extension into the Gemini CLI:

```bash
gemini extensions install https://github.com/magic-meal-kits/mmk-skills
```

Installing this extension gives your Gemini CLI agent direct access to all `mmk` commands and Magic Meal Kits agent skills. Because `mmk` handles its own authentication, you simply need to authenticate your terminal once (Step 1) and the extension will automatically inherit your credentials.

#### Claude Code / OpenCode / Codex / Cursor

```bash
npx skills add https://github.com/magic-meal-kits/mmk-skills
```

<details>
<summary>More install options (by service, individual sub-commands, plugin marketplace)</summary>

```bash
# Install by service (root + all sub-commands)
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-notion
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-paymint
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-threads
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-youtube
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-plaud

# Or install individual sub-commands only
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-notion-page
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-paymint-send
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-threads-posts
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-youtube-transcript
npx skills add https://github.com/magic-meal-kits/mmk-skills/tree/main/skills/mmk-plaud-file
```

```bash
# Or install via plugin marketplace
/plugin marketplace add magic-meal-kits/mmk-skills
/plugin install mmk-skills@mmk-plugins
```

</details>

### 3. Use

Type `/` in Claude Code to see available skills:

```
/mmk-notion             — Notion commands overview (links to sub-commands)
/mmk-notion-page        — Page invite, revoke, publish, config, duplicate, markdown get/create/append/replace/update, transcript, get, update (16 commands)
/mmk-notion-workspace   — Workspace invite and remove (2 commands)
/mmk-notion-team        — Team list, invite, remove (3 commands)
/mmk-notion-subscription — Subscription details (1 command)
/mmk-notion-people      — List members/guests, guest pages (2 commands)
/mmk-notion-database    — Schema, query, insert, update, upsert, delete, AI summary, AI summary auto (8 commands)
/mmk-notion-comment     — Page comments: add, list, reply (3 commands)
/mmk-notion-user        — User details, list workspace users (2 commands)
/mmk-notion-meeting     — AI meeting notes (1 command)
/mmk-notion-emoji       — Custom workspace emojis (1 command)
/mmk-notion-view        — Database views: list, get, create, update, delete (5 commands)
/mmk-paymint            — Paymint overview + license resolution + errors
/mmk-paymint-licenses   — List licenses
/mmk-paymint-send       — Send invoice
/mmk-paymint-status     — Check invoice status
/mmk-paymint-cancel     — Cancel/refund invoice
/mmk-paymint-resend     — Resend invoice SMS
/mmk-paymint-bulk-send  — Bulk send invoices
/mmk-paymint-bulk-resend — Bulk resend SMS
/mmk-threads            — Threads overview + pagination
/mmk-threads-posts      — Get recent posts
/mmk-threads-insights   — Account analytics
/mmk-threads-replies    — Post replies/conversation
/mmk-youtube            — YouTube overview + positional arg note
/mmk-youtube-metadata   — Video metadata
/mmk-youtube-videotype  — Video vs Short
/mmk-youtube-transcript — Video transcript
/mmk-plaud              — Plaud overview + credential overrides (25 commands)
/mmk-plaud-file         — File list/get, audio URL, download, transcript, summary, transsumm, rename, organize (9 commands)
/mmk-plaud-search       — Search transcripts and filenames (1 command)
/mmk-plaud-user         — Plaud user profile (1 command)
/mmk-plaud-doctor       — Connectivity check (1 command)
/mmk-plaud-admin        — Low-level passthroughs: ai, combine, config, device, file, membership, search, tag, trans-quota, user (13 commands)
/mmk-notion-onboard     — Invite to Notion with Gmail fallback (recipe)
/mmk-notion-database-ai-summary — Bulk AI summary across database pages (recipe)
/mmk-notion-meeting-comment — Post structured meeting summary as page comment (recipe)
/mmk-paymint-notion-schema  — Recommended Notion DB schema for Paymint invoices
/mmk-paymint-notion-invoice — Batch send Paymint invoices from Notion database (recipe)
/mmk-youtube-channel-summary — Summarize YouTube channel videos into Notion database (recipe)
```

## Skill Inventory

| Skill | Type | Trigger | Commands |
|-------|------|---------|----------|
| `mmk-shared` | Background | Auto-loaded | Foundation: auth, flags, errors |
| **Notion** | | | |
| `mmk-notion` | Root | `/mmk-notion` | Overview + sub-command links (44 commands total) |
| `mmk-notion-page` | Sub-command | `/mmk-notion-page` | invite, revoke, publish, unpublish, config, publish-settings, duplicate, list-published, markdown get/create/append/replace/update, transcript, get, update |
| `mmk-notion-workspace` | Sub-command | `/mmk-notion-workspace` | invite, remove |
| `mmk-notion-team` | Sub-command | `/mmk-notion-team` | list, invite, remove |
| `mmk-notion-subscription` | Sub-command | `/mmk-notion-subscription` | subscription |
| `mmk-notion-people` | Sub-command | `/mmk-notion-people` | list, guest-pages |
| `mmk-notion-database` | Sub-command | `/mmk-notion-database` | schema, query, insert, update, upsert, delete, ai-summary, ai-summary-auto |
| `mmk-notion-comment` | Sub-command | `/mmk-notion-comment` | add, list, reply |
| `mmk-notion-user` | Sub-command | `/mmk-notion-user` | get, list |
| `mmk-notion-meeting` | Sub-command | `/mmk-notion-meeting` | list |
| `mmk-notion-emoji` | Sub-command | `/mmk-notion-emoji` | list |
| `mmk-notion-view` | Sub-command | `/mmk-notion-view` | list, get, create, update, delete |
| **Paymint** | | | |
| `mmk-paymint` | Root | `/mmk-paymint` | Overview + license resolution + errors (7 commands total) |
| `mmk-paymint-licenses` | Sub-command | `/mmk-paymint-licenses` | licenses |
| `mmk-paymint-send` | Sub-command | `/mmk-paymint-send` | send |
| `mmk-paymint-status` | Sub-command | `/mmk-paymint-status` | status |
| `mmk-paymint-cancel` | Sub-command | `/mmk-paymint-cancel` | cancel |
| `mmk-paymint-resend` | Sub-command | `/mmk-paymint-resend` | resend |
| `mmk-paymint-bulk-send` | Sub-command | `/mmk-paymint-bulk-send` | bulk-send |
| `mmk-paymint-bulk-resend` | Sub-command | `/mmk-paymint-bulk-resend` | bulk-resend |
| **Threads** | | | |
| `mmk-threads` | Root | `/mmk-threads` | Overview + pagination (3 commands total) |
| `mmk-threads-posts` | Sub-command | `/mmk-threads-posts` | posts |
| `mmk-threads-insights` | Sub-command | `/mmk-threads-insights` | insights |
| `mmk-threads-replies` | Sub-command | `/mmk-threads-replies` | replies |
| **YouTube** | | | |
| `mmk-youtube` | Root | `/mmk-youtube` | Overview + positional arg note (3 commands total) |
| `mmk-youtube-metadata` | Sub-command | `/mmk-youtube-metadata` | metadata |
| `mmk-youtube-videotype` | Sub-command | `/mmk-youtube-videotype` | videotype |
| `mmk-youtube-transcript` | Sub-command | `/mmk-youtube-transcript` | transcript |
| **Plaud** | | | |
| `mmk-plaud` | Root | `/mmk-plaud` | Overview + credential overrides (25 commands total) |
| `mmk-plaud-file` | Sub-command | `/mmk-plaud-file` | list, get, audio-url, download, transcript, summary, transsumm, rename, organize |
| `mmk-plaud-search` | Sub-command | `/mmk-plaud-search` | search |
| `mmk-plaud-user` | Sub-command | `/mmk-plaud-user` | me |
| `mmk-plaud-doctor` | Sub-command | `/mmk-plaud-doctor` | doctor |
| `mmk-plaud-admin` | Sub-command | `/mmk-plaud-admin` | ai, combine, config, device, file, membership, search, tag, trans-quota, user (13 commands) |
| **Recipes** | | | |
| `mmk-notion-onboard` | Recipe | Manual only | Multi-step: Notion invite + Gmail signup fallback |
| `mmk-notion-database-ai-summary` | Recipe | Manual only | Multi-step: Discover AI config + batch summary generation |
| `mmk-notion-meeting-comment` | Recipe | Manual only | Multi-step: Meeting transcript + structured comment |
| `mmk-paymint-notion-schema` | Reference | `/mmk-paymint-notion-schema` | Recommended Notion DB schema for Paymint invoices |
| `mmk-paymint-notion-invoice` | Recipe | Manual only | Multi-step: Notion DB query + batch Paymint invoice send |
| `mmk-youtube-channel-summary` | Recipe | Manual only | Multi-step: RSS fetch + filter Shorts + metadata/transcript + AI summary + Notion upsert |

### Architecture

```
mmk-shared (background)              <- Foundation: auth, flags, errors
├── mmk-notion (root)                <- Overview + tips (44 commands)
│   ├── mmk-notion-page              <- 16 page commands
│   ├── mmk-notion-workspace         <- 2 workspace commands
│   ├── mmk-notion-team              <- 3 team commands
│   ├── mmk-notion-subscription      <- 1 subscription command
│   ├── mmk-notion-people            <- 2 people commands
│   ├── mmk-notion-database          <- 8 database commands
│   ├── mmk-notion-comment           <- 3 comment commands
│   ├── mmk-notion-user              <- 2 user commands
│   ├── mmk-notion-meeting           <- 1 meeting command
│   ├── mmk-notion-emoji             <- 1 emoji command
│   └── mmk-notion-view              <- 5 view commands
├── mmk-paymint (root)               <- Overview + license resolution + errors (7 commands)
│   ├── mmk-paymint-licenses         <- List licenses
│   ├── mmk-paymint-send             <- Send invoice
│   ├── mmk-paymint-status           <- Check status
│   ├── mmk-paymint-cancel           <- Cancel/refund
│   ├── mmk-paymint-resend           <- Resend SMS
│   ├── mmk-paymint-bulk-send        <- Bulk send invoices
│   └── mmk-paymint-bulk-resend      <- Bulk resend SMS
├── mmk-threads (root)               <- Overview + pagination (3 commands)
│   ├── mmk-threads-posts            <- Get posts
│   ├── mmk-threads-insights         <- Account analytics
│   └── mmk-threads-replies          <- Post replies
├── mmk-youtube (root)               <- Overview + positional args (3 commands)
│   ├── mmk-youtube-metadata         <- Video metadata
│   ├── mmk-youtube-videotype        <- Video vs Short
│   └── mmk-youtube-transcript       <- Video transcript
├── mmk-plaud (root)                 <- Plaud overview + credential overrides (25 commands)
│   ├── mmk-plaud-file               <- 9 file commands (list, get, audio-url, download, transcript, summary, transsumm, rename, organize)
│   ├── mmk-plaud-search             <- Search transcripts and filenames
│   ├── mmk-plaud-user               <- Plaud user profile
│   ├── mmk-plaud-doctor             <- Connectivity check (token + user + devices)
│   └── mmk-plaud-admin              <- 13 low-level passthroughs (ai, combine, config, device, file, membership, search, tag, trans-quota, user)
├── mmk-notion-onboard (recipe)      <- Notion invite + Gmail fallback
├── mmk-notion-database-ai-summary (recipe) <- Bulk AI summary for database
├── mmk-notion-meeting-comment (recipe) <- Meeting transcript + summary comment
├── mmk-paymint-notion-schema (reference) <- Recommended Notion DB schema for invoices
├── mmk-paymint-notion-invoice (recipe) <- Batch invoice from Notion DB
└── mmk-youtube-channel-summary (recipe) <- YouTube channel summary to Notion
```

- **Background skills** load automatically when relevant context is detected
- **Root skills** provide overview, shared guidance, and links to sub-command skills
- **Sub-command skills** are independently installable and contain focused command docs
- **Recipe skills** require manual invocation (`disable-model-invocation: true`) because they have side effects

## Usage Examples

### Notion: Invite a user to a page

```
/mmk-notion-page
> Invite kim@example.com as editor to https://notion.so/my-page-abc123
```

### Paymint: Send an invoice

```
/mmk-paymint-send
> Send a 50,000 KRW invoice to 01012345678 for "Workshop Fee", name "Kim", message "March workshop", expiring 2026-04-01
```

### Threads: Check recent posts with engagement

```
/mmk-threads-posts
> Show my last 10 posts with engagement metrics
```

### Paymint: Batch invoices from Notion

```
/mmk-paymint-notion-invoice
> Send invoices from database abc123-def456
```

### YouTube: Get a transcript

```
/mmk-youtube-transcript
> Get the transcript for https://youtube.com/watch?v=abc123
```

### YouTube: Summarize channel videos to Notion

```
/mmk-youtube-channel-summary
> Channels: UC_x5XG1OV2P6uZZ5FSM9Ttw, UCddiUEpeqJcYeBxX1IVBKvQ
> Database: abc123-def456, last 7 days
```

### Plaud: List recent recordings with transcripts and summaries

```
/mmk-plaud-file
> List my 10 most recent recordings with transcripts and summaries attached
```

## Compatibility

These skills work with Claude Code, Gemini CLI, OpenCode, Codex, Cursor, and [38+ agents](https://github.com/nichochar/skills). They also work alongside [GWS Skills](https://github.com/pureugong/gws) (Google Workspace CLI). The `mmk-notion-onboard` skill composes both `mmk` and `gws` commands.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [MMK CLI](https://www.npmjs.com/package/@magic-meal-kits/cli) (`npm install -g @magic-meal-kits/cli@beta`)
- Active MMK server endpoint
