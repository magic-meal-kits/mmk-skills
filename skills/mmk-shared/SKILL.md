---
name: mmk-shared
description: This skill provides background knowledge for all mmk CLI operations. It should be loaded automatically when the user mentions "mmk", "magic meal kits cli", "notion invite", "notion revoke", "paymint", or any mmk command. Covers prerequisites, authentication, global flags, and error handling for the mmk CLI tool.
---

# MMK CLI — Shared Foundation

Background knowledge for all skills that use the `mmk` CLI.

## Prerequisites

The `mmk` CLI must be installed and configured:

```bash
# Install
npm install -g @magic-meal-kits/cli@beta

# Configure server endpoint
mmk config set server <server-url>

# Authenticate
mmk auth login
```

## Global Flags

| Flag | Short | Default | Purpose |
|------|-------|---------|---------|
| `--output` | `-o` | `table` | Output format: `table` or `json` |
| `--server` | | (from config) | Override server URL |
| `--config` | | `~/.config/mmk/config.yaml` | Override config file |

**Critical for skill composition**: Always use `-o json` when composing multiple commands or parsing output programmatically. Table output is for human display only.

## Error Format

When a command fails, the CLI outputs to stderr:

```
✗ HTTP <status_code>
  <error message>
```

With `-o json`, errors are returned as:
```json
{"error": "<error message>"}
```

The error message comes from the server's JSON response `{"error": "..."}` field. The CLI exits with a non-zero status code on failure.

### Common Error Patterns

| Error | Meaning | Action |
|-------|---------|--------|
| `"not authenticated"` or auth failure | Token expired or not logged in | Run `mmk auth login` |
| `"request failed with status 400"` | Bad request (missing/invalid params) | Check required flags |
| `"request failed with status 403"` | Forbidden | Check permissions |
| `"request failed with status 404"` | Resource not found | Verify block-id/email |

## Auth Troubleshooting

If commands return authentication errors:
1. Run `mmk auth status` to check current auth state
2. Run `mmk auth login` to re-authenticate
3. Verify server URL: `mmk config get server`

## Command Reference

Top-level commands available via `mmk`:
- [`mmk notion`](../mmk-notion/SKILL.md) — Notion page, workspace, team, people, database, meeting management (24 commands)
- [`mmk paymint`](../mmk-paymint/SKILL.md) — Payment/invoice management (7 commands)
- [`mmk threads`](../mmk-threads/SKILL.md) — Threads account data: posts, insights, replies (3 commands)
- [`mmk youtube`](../mmk-youtube/SKILL.md) — YouTube tools: transcript, metadata, video type (3 commands)
- `mmk auth` — Authentication
- `mmk config` — CLI configuration
- `mmk doctor` — Check CLI setup and server connectivity
- `mmk plan` — Show current plan and feature access
- `mmk status` — Show server health and connection status
- `mmk upgrade` — Check for and install CLI updates (`--check` for check-only mode)
