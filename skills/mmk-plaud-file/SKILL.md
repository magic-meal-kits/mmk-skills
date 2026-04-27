---
name: mmk-plaud-file
description: Manage Plaud recordings — list, get detail, audio URL, download, rename, organize into folders, fetch transcript/summary, trigger auto transcript+summary. Triggers on "plaud file", "plaud recordings", "list plaud files", "plaud transcript", "plaud summary", "plaud audio url", "download plaud audio", "download recording", "rename plaud recording", "organize plaud", "plaud transsumm", "auto transcript summary".
allowed-tools: Bash(mmk *)
---

# MMK Plaud — File Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-plaud`](../mmk-plaud/SKILL.md) — includes credential overrides and requirements.

Always use `-o json` when parsing results or composing with other commands.

---

## list — List recordings (compose transcripts/summaries)

```bash
mmk plaud file list --limit 10 -o json
mmk plaud file list --with transcripts,summaries --limit 25 -o json
mmk plaud file list --sort start_time --asc --trash active -o json
```

**Required:** none

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--limit` | `20` | Max files to return (server caps at 99999) |
| `--skip` | `0` | Offset for pagination |
| `--sort` | `edit_time` | Sort field: `edit_time` or `start_time` |
| `--asc` | (descending) | Sort ascending |
| `--trash` | `active` | Filter: `active`, `trash`, or `all` |
| `--with` | | Compose attachments: comma list of `transcripts`, `summaries`, `shares`, `audio-urls` |
| `--concurrency` | `4` | Max concurrent attachment fetches per file (1–8); only with `--with` |
| `--since` | `0` | Only files with `start_time >= this epoch-ms` (only with `--with`) |
| `--strict` | | Fail-fast on any attachment error (only with `--with`) |

---

## get — Get file detail (compose transcript/summary)

```bash
mmk plaud file get <file_id> -o json
mmk plaud file get <file_id> --with transcripts,summaries -o json
```

**Required:** `<file_id>` (positional)

**Optional:**

| Flag | Description |
|------|-------------|
| `--with` | Compose attachments: comma list of `transcripts`, `summaries`, `shares`, `audio-urls` |
| `--strict` | Fail-fast on any attachment error (only with `--with`) |

---

## audio-url — Get pre-signed audio URL (when available)

```bash
mmk plaud file audio-url <file_id> -o json
```

**Required:** `<file_id>` (positional)

---

## download — Download the audio file to disk via the signed URL

```bash
mmk plaud file download <file_id>
mmk plaud file download <file_id> --out ./recordings/
mmk plaud file download <file_id> --out my-note.ogg --force
mmk plaud file download <file_id> --quiet
```

**Required:** `<file_id>` (positional)

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--out` | current directory (auto-named) | Output file or directory path |
| `--force` | false | Overwrite if the target file already exists |
| `--quiet` | false | Suppress progress output; print only the saved path |

Default filename: `{file_name}.{ext}` where `file_name` comes from the recording's metadata (non-filesystem-safe characters stripped) and `ext` is parsed from the signed URL path. Falls back to `{file_id}.ogg` when the name is empty.

The file is streamed directly from S3 to disk — the server only returns the signed URL and does not proxy the bytes.

---

## transcript — Get decoded transcript JSON

```bash
mmk plaud file transcript <file_id> -o json
```

**Required:** `<file_id>` (positional)

---

## summary — Get AI-generated summary (Markdown)

```bash
mmk plaud file summary <file_id> -o json
```

**Required:** `<file_id>` (positional)

Returns a Markdown summary as a string in the JSON payload.

---

## transsumm — Trigger auto transcript + summary (optionally wait)

```bash
mmk plaud file transsumm <file_id> -o json
mmk plaud file transsumm <file_id> --wait --timeout 120 -o json
mmk plaud file transsumm <file_id> --llm gemini-3.1-pro --language ko --no-diarization -o json
```

**Required:** `<file_id>` (positional)

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--wait` | `false` | Poll until the task completes and print transcript + summary |
| `--timeout` | `300` | Server-side timeout in seconds (only with `--wait`) |
| `--poll-interval` | `2` | Server-side poll interval in seconds (only with `--wait`) |
| `--check-quota` | off / on with `--wait` | Pre-check remaining seconds vs file duration |
| `--reload` | `false` | Re-run transcription even if already transcribed |
| `--diarization` | `true` | Enable speaker diarization (pass `--diarization=false` to disable) |
| `--language` | `auto` | Transcription language (server picks if `auto`) |
| `--llm` | `auto` | Summary LLM model (e.g. `gpt-5.2`, `gemini-3.1-pro`) |
| `--summ-type` | `AUTO-SELECT` | Summary template id |
| `--timezone` | local offset | Timezone offset hours |

Without `--wait` the command returns task handles; with `--wait` it blocks until completion and returns the finished transcript + summary.

---

## rename — Rename a Plaud recording

```bash
mmk plaud file rename <file_id> "Team standup 2026-04-14" -o json
```

**Required:** `<file_id>` (positional), new name (positional)

---

## organize — Put recordings into a folder (creates the folder if missing)

```bash
mmk plaud file organize --files f1,f2 --into "Interviews" -o json
mmk plaud file organize --files f1 --into "Q2 Planning" --color "#fb5c5c" --icon e627 -o json
mmk plaud file organize --files f1,f2 --into "interviews" --match fold -o json
mmk plaud file organize --files f1 --into "Must Exist" --no-create -o json
```

**Required:** `--files` (comma-separated file IDs), `--into` (target folder name)

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--match` | `exact` | Tag name match mode: `exact` or `fold` (case/unicode-insensitive) |
| `--no-create` | | Fail instead of creating a missing folder (useful for strict automation) |
| `--color` | | Color for the new folder if it has to be created |
| `--icon` | | Icon glyph for the new folder if it has to be created |

Prefer `organize` over `mmk plaud admin tag …` primitives — it finds-or-creates the tag and assigns files in one call.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-plaud](../mmk-plaud/SKILL.md) — All Plaud commands overview, credential overrides
- [mmk-plaud-search](../mmk-plaud-search/SKILL.md) — Search transcripts and filenames
- [mmk-plaud-admin](../mmk-plaud-admin/SKILL.md) — Low-level passthroughs (AI status, tag primitives, file share/ack-pull)
