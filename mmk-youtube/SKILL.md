---
name: mmk-youtube
description: This skill should be used when the user asks to get YouTube video metadata, transcripts, or determine video type (video vs Short) using the mmk CLI. Triggers on phrases like "youtube transcript", "video metadata", "is it a short", "video type", "get transcript", "youtube info".
allowed-tools: Bash(mmk *)
---

# MMK YouTube

Get YouTube video metadata, transcripts, and video type using the `mmk` CLI.

Always use `-o json` when parsing results or composing with other commands.

All commands accept a **positional argument** — either a full YouTube URL or a plain video ID. The CLI auto-detects the format (presence of `/` or `.` indicates URL).

---

## Commands

### metadata — Get video metadata

```bash
mmk youtube metadata <video_url_or_id> -o json
mmk youtube metadata "https://www.youtube.com/watch?v=dQw4w9WgXcQ" -o json
mmk youtube metadata dQw4w9WgXcQ -o json
```

Returns: title, author, channel handle, subscriber count, view count, like count, publish date.

### videotype — Identify video vs Short

```bash
mmk youtube videotype <video_url_or_id> -o json
```

Returns: video ID, type (`video` or `short`), source URL.

### transcript — Get video transcript

```bash
mmk youtube transcript <video_url_or_id> -o json
mmk youtube transcript <video_url_or_id> --format srt
mmk youtube transcript <video_url_or_id> --format vtt
```

**Optional:** `--format` — `text` (default), `json`, `srt`, `vtt`

In table mode, prints raw transcript text. In JSON mode, returns structured response with `content`, `video_id`, and `format` fields.
