---
name: mmk-plaud-admin
description: Advanced low-level Plaud API passthroughs — AI task status, combine tasks, config flags, device list, file ack-pull/share, membership status, search history, tag primitives, transcription quota, user feature access. Triggers on "plaud admin", "plaud ai status", "plaud combine", "plaud config", "plaud device list", "plaud ack-pull", "plaud share", "plaud membership", "plaud search history", "plaud tag", "plaud trans-quota", "plaud feature access".
allowed-tools: Bash(mmk *)
---

# MMK Plaud — Admin Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-plaud`](../mmk-plaud/SKILL.md) — includes credential overrides and requirements.

Always use `-o json` when parsing results or composing with other commands.

This subtree groups raw 1:1 passthroughs to upstream Plaud API endpoints. Most users should prefer the user-facing verbs under [`mmk-plaud-file`](../mmk-plaud-file/SKILL.md) and [`mmk-plaud-search`](../mmk-plaud-search/SKILL.md). Use `admin` only for power-user scripting, troubleshooting, or features not yet exposed by the higher-level commands.

---

## ai status — Poll AI processing status for one or more files

```bash
mmk plaud admin ai status --file-ids 9eb5d6e8...,abc123... -o json
```

**Required:** `--file-ids` (comma-separated or repeatable)

---

## combine running — List in-flight combine tasks for the current user

```bash
mmk plaud admin combine running -o json
```

**Required:** none

Lists audio combine / merge tasks that are currently in-flight.

---

## config — Show Plaud runtime feature flags (`/config/init`)

```bash
mmk plaud admin config -o json
```

**Required:** none

Returns the Plaud client feature-flag bundle. Useful for debugging feature gating discrepancies.

---

## device list — List connected Plaud hardware devices

```bash
mmk plaud admin device list -o json
```

**Required:** none

---

## file ack-pull — Acknowledge that a file was pulled from the Plaud device

```bash
mmk plaud admin file ack-pull <file_id> -o json
```

**Required:** `<file_id>` (positional)

Low-level device sync primitive — normally the Plaud client/app handles this.

---

## file share — Get private + public share state for a file

```bash
mmk plaud admin file share <file_id> -o json
```

**Required:** `<file_id>` (positional)

Returns both private and public share metadata in one call.

---

## membership status — Show free-trial / membership billing status

```bash
mmk plaud admin membership status -o json
```

**Required:** none

---

## search history — Show recent search queries

```bash
mmk plaud admin search history -o json
mmk plaud admin search history --limit 20 -o json
```

**Required:** none

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | `10` | Max history entries |

---

## tag list — List file tags (folders)

```bash
mmk plaud admin tag list -o json
```

**Required:** none

---

## tag create — Create a new tag (folder)

```bash
mmk plaud admin tag create --name "Interviews" --color "#46cf6c" --icon e627 -o json
```

**Required:** `--name`

**Optional:**

| Flag | Description |
|------|-------------|
| `--color` | Display color, e.g. `#46cf6c` |
| `--icon` | Icon glyph code, e.g. `e627` |

---

## tag assign — Assign or clear a tag on one or more files (by tag id)

```bash
mmk plaud admin tag assign --files f1,f2 --tag-id abc123 -o json
mmk plaud admin tag assign --files f1 --tag-id "" -o json
```

**Required:** `--files` (comma-separated), `--tag-id` (empty string clears)

Prefer `mmk plaud file organize` for folder assignment — it handles tag lookup/creation and file assignment in one call.

---

## trans-quota — Transcription seconds remaining + starter/usage flags

```bash
mmk plaud admin trans-quota -o json
```

**Required:** none

Combined view of remaining transcription quota and related billing flags.

---

## user feature-access — Show feature flags available to the current user

```bash
mmk plaud admin user feature-access -o json
```

**Required:** none

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-plaud](../mmk-plaud/SKILL.md) — All Plaud commands overview, credential overrides
- [mmk-plaud-file](../mmk-plaud-file/SKILL.md) — High-level file operations (prefer over admin file primitives)
- [mmk-plaud-search](../mmk-plaud-search/SKILL.md) — High-level transcript/filename search
- [mmk-plaud-doctor](../mmk-plaud-doctor/SKILL.md) — Token + user + devices check in one call
