---
name: mmk-youtube-transcript
description: Get YouTube video transcript in text, JSON, SRT, or VTT format. Triggers on "youtube transcript", "get transcript", "video transcript", "subtitles", "captions".
allowed-tools: Bash(mmk *)
---

# MMK YouTube — Transcript

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-youtube`](../mmk-youtube/SKILL.md) — includes positional argument note.

Always use `-o json` when parsing results or composing with other commands.

All commands accept a **positional argument** — either a full YouTube URL or a plain video ID. The CLI auto-detects the format.

---

## transcript — Get video transcript

```bash
mmk youtube transcript <video_url_or_id> -o json
mmk youtube transcript <video_url_or_id> --format srt
mmk youtube transcript <video_url_or_id> --format vtt
```

**Optional:** `--format` — `text` (default), `json`, `srt`, `vtt`

In table mode, prints raw transcript text. In JSON mode, returns structured response with `content`, `video_id`, and `format` fields.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-youtube](../mmk-youtube/SKILL.md) — All YouTube commands overview
- [mmk-youtube-metadata](../mmk-youtube-metadata/SKILL.md) — Get video metadata
- [mmk-youtube-videotype](../mmk-youtube-videotype/SKILL.md) — Identify video vs Short
