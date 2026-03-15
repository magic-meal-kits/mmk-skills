---
name: mmk-youtube-videotype
description: Identify whether a YouTube video is a regular video or a Short. Triggers on "is it a short", "video type", "video or short", "check video type".
allowed-tools: Bash(mmk *)
---

# MMK YouTube — Video Type

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-youtube`](../mmk-youtube/SKILL.md) — includes positional argument note.

Always use `-o json` when parsing results or composing with other commands.

All commands accept a **positional argument** — either a full YouTube URL or a plain video ID. The CLI auto-detects the format.

---

## videotype — Identify video vs Short

```bash
mmk youtube videotype <video_url_or_id> -o json
```

Returns: video ID, type (`video` or `short`), source URL.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-youtube](../mmk-youtube/SKILL.md) — All YouTube commands overview
- [mmk-youtube-metadata](../mmk-youtube-metadata/SKILL.md) — Get video metadata
- [mmk-youtube-transcript](../mmk-youtube-transcript/SKILL.md) — Get video transcript
