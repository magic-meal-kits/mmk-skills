---
name: mmk-plaud
description: This skill should be used when the user asks to list Plaud recordings, fetch transcripts/summaries, organize recordings into folders, search transcripts, inspect Plaud devices, check membership/quota, or any Plaud voice recorder operation using the mmk CLI. Triggers on phrases like "plaud", "plaud recordings", "plaud transcript", "plaud summary", "plaud search", "plaud folder", "organize recordings", "plaud doctor", "plaud quota", "plaud membership", "plaud devices".
allowed-tools: Bash(mmk *)
---

# MMK Plaud

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

Read-only access to the unofficial Plaud web API using the `mmk` CLI. Covers recording listing, transcript and summary retrieval, folder organization, transcript search, and admin diagnostics.

Always use `-o json` when parsing results or composing with other commands.

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `file` | List/inspect recordings, get audio URL, download, rename, organize into folders, fetch transcripts/summaries, list/get content tabs, trigger auto transcript+summary (11 commands) | [`mmk-plaud-file`](../mmk-plaud-file/SKILL.md) |
| `search` | Search transcripts and filenames (1 command) | [`mmk-plaud-search`](../mmk-plaud-search/SKILL.md) |
| `user` | Show current Plaud user profile (1 command) | [`mmk-plaud-user`](../mmk-plaud-user/SKILL.md) |
| `doctor` | Connectivity check: validates token and lists user/devices (1 command) | [`mmk-plaud-doctor`](../mmk-plaud-doctor/SKILL.md) |
| `admin` | Advanced 1:1 API passthroughs: AI status, combine tasks, config, devices, file ack/share, membership, search history, tags, trans-quota, feature access (13 commands) | [`mmk-plaud-admin`](../mmk-plaud-admin/SKILL.md) |

## Requirements

- **Pro license** required on the MMK server (plaud commands check this on every call).
- A valid Plaud bearer token is loaded by default from Google Cloud Secret Manager on the server side.

## Plaud Credential Overrides

All `mmk plaud …` commands accept two shared flags for ad-hoc overrides or testing:

| Flag | Env var | Purpose |
|------|---------|---------|
| `--plaud-token` | `MMK_PLAUD_TOKEN` | Override Plaud bearer token (format: `bearer eyJ...`) |
| `--plaud-api-domain` | `MMK_PLAUD_API_DOMAIN` | Override Plaud regional API domain (e.g. `https://api-apne1.plaud.ai`) |

When omitted, the server falls back to its configured secret.

## High-level vs Low-level

Prefer the user-facing verbs under `mmk plaud file …` and `mmk plaud search`. The `mmk plaud admin …` subtree is a 1:1 passthrough to upstream Plaud endpoints — use it only for power-user scripting, troubleshooting, or features not yet exposed by the higher-level commands.

For folder assignment, prefer `mmk plaud file organize` (which finds-or-creates the tag and assigns files in one call) over the lower-level `mmk plaud admin tag …` primitives.

## Tips

1. `mmk plaud doctor` is the fastest way to verify token + connectivity before scripting.
2. `mmk plaud file list --with transcripts,summaries` composes attachments in one call — use `--concurrency` (1–8) to tune parallelism and `--strict` to fail-fast on any attachment error.
3. `mmk plaud file transsumm --wait` polls the server until the transcript+summary task completes, returning both artifacts together.
4. Always use `-o json` for reliable parsing in scripts and skill compositions.
