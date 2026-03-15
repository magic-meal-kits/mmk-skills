---
name: mmk-youtube
description: This skill should be used when the user asks to get YouTube video metadata, transcripts, or determine video type (video vs Short) using the mmk CLI. Triggers on phrases like "youtube transcript", "video metadata", "is it a short", "video type", "get transcript", "youtube info".
allowed-tools: Bash(mmk *)
---

# MMK YouTube

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

Get YouTube video metadata, transcripts, and video type using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

All commands accept a **positional argument** — either a full YouTube URL or a plain video ID. The CLI auto-detects the format (presence of `/` or `.` indicates URL).

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `metadata` | Get video metadata (title, author, views, likes, publish date) | [`mmk-youtube-metadata`](../mmk-youtube-metadata/SKILL.md) |
| `videotype` | Identify video vs Short | [`mmk-youtube-videotype`](../mmk-youtube-videotype/SKILL.md) |
| `transcript` | Get video transcript (text, JSON, SRT, VTT) | [`mmk-youtube-transcript`](../mmk-youtube-transcript/SKILL.md) |
