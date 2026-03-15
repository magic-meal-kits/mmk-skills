---
name: mmk-youtube-metadata
description: Get YouTube video metadata — title, author, views, likes, publish date. Triggers on "video metadata", "youtube info", "video details", "video stats".
allowed-tools: Bash(mmk *)
---

# MMK YouTube — Metadata

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-youtube`](../mmk-youtube/SKILL.md) — includes positional argument note.

Always use `-o json` when parsing results or composing with other commands.

All commands accept a **positional argument** — either a full YouTube URL or a plain video ID. The CLI auto-detects the format.

---

## metadata — Get video metadata

```bash
mmk youtube metadata <video_url_or_id> -o json
mmk youtube metadata "https://www.youtube.com/watch?v=dQw4w9WgXcQ" -o json
mmk youtube metadata dQw4w9WgXcQ -o json
```

Returns: title, author, channel handle, subscriber count, view count, like count, publish date.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-youtube](../mmk-youtube/SKILL.md) — All YouTube commands overview
- [mmk-youtube-transcript](../mmk-youtube-transcript/SKILL.md) — Get video transcript
- [mmk-youtube-videotype](../mmk-youtube-videotype/SKILL.md) — Identify video vs Short
