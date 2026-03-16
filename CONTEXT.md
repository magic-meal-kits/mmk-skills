# Magic Meal Kits CLI (`mmk`) Context

CLI tool for automating Notion, Paymint, Threads, and YouTube operations.

## Rules of Engagement for Agents

- Always use `-o json` when parsing results or composing commands
- Check `mmk auth status` if you get authentication errors
- Use `--help` at any level to discover commands and flags
- Table output is for human display only — always parse JSON

## Setup

```bash
# Install
npm install -g @magic-meal-kits/cli@beta

# Configure server endpoint
mmk config set server <server-url>

# Authenticate
mmk auth login
```

## Core Syntax

```
mmk <service> [sub-command] <command> [flags]
```

## Key Flags

| Flag | Short | Default | Purpose |
|------|-------|---------|---------|
| `--output` | `-o` | `table` | Output format: `table` or `json` |
| `--server` | | (from config) | Override server URL |
| `--config` | | `~/.config/mmk/config.yaml` | Override config file |

## Services

| Service | Commands | Description |
|---------|----------|-------------|
| `mmk notion` | 20 | Page, workspace, team, people, database, meeting management |
| `mmk paymint` | 5 | Payment/invoice management (send, cancel, status, resend, licenses) |
| `mmk threads` | 3 | Threads account data: posts, insights, replies |
| `mmk youtube` | 3 | YouTube tools: transcript, metadata, video type |

## Error Format

When a command fails, the CLI outputs to stderr:

```
✗ HTTP <status_code>
  <error message>
```

With `-o json`, errors return: `{"error": "<error message>"}`

### Common Errors

| Error | Meaning | Action |
|-------|---------|--------|
| `"not authenticated"` | Token expired or not logged in | Run `mmk auth login` |
| `"request failed with status 400"` | Bad request | Check required flags |
| `"request failed with status 403"` | Forbidden | Check permissions |
| `"request failed with status 404"` | Resource not found | Verify IDs/emails |

## Utility Commands

| Command | Description |
|---------|-------------|
| `mmk auth login` | Authenticate with server |
| `mmk auth status` | Check current auth state |
| `mmk config set <key> <value>` | Set configuration |
| `mmk config get <key>` | Get configuration |
| `mmk doctor` | Check CLI setup and server connectivity |
| `mmk plan` | Show current plan and feature access |
| `mmk status` | Show server health and connection status |
| `mmk upgrade` | Check for and install CLI updates (`--check` for check-only) |
