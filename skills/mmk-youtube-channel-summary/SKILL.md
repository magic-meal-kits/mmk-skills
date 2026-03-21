---
name: mmk-youtube-channel-summary
description: Fetch recent YouTube channel videos via RSS, filter out Shorts, get metadata and transcripts, AI-summarize, and upsert into a Notion database. Supports multiple channels. Triggers on "youtube channel summary", "channel summary to notion", "youtube rss to notion", "summarize youtube channel", "youtube channel notion sync".
disable-model-invocation: true
allowed-tools: Bash(mmk *), Bash(curl *)
---

# Recipe: YouTube Channel Summary to Notion

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Uses: [`mmk-youtube-videotype`](../mmk-youtube-videotype/SKILL.md) — Filter out Shorts.
> Uses: [`mmk-youtube-metadata`](../mmk-youtube-metadata/SKILL.md) — Video metadata (views, likes, etc.).
> Uses: [`mmk-youtube-transcript`](../mmk-youtube-transcript/SKILL.md) — Video transcript for summarization.
> Uses: [`mmk-notion-database`](../mmk-notion-database/SKILL.md) — Schema discovery and upsert commands.

A fully automatic workflow that fetches recent videos from one or more YouTube channel RSS feeds, filters out Shorts, collects metadata and transcripts, generates AI summaries, and upserts everything into a Notion database.

**This recipe has side effects** (creates/updates records in a Notion database). It requires manual invocation only.

**No confirmation prompts** — the recipe runs end-to-end automatically once invoked with the required parameters.

## Database Schema Requirements

The recipe discovers properties by their **Notion type and semantic meaning**, not by exact name. English and Korean property names are both supported automatically.

### Required Properties

| Purpose | Notion Type | Maps to | Auto-match names |
|---------|-------------|---------|------------------|
| VideoID | title | Upsert lookup key | (title property — always unambiguous) |
| Title | rich_text | Video title | Title, 제목, 영상제목 |
| Channel | rich_text | Channel name | Channel, 채널, 채널명 |
| URL | url | Video URL | URL, 링크, 영상링크 |
| Published | date | Publish date | Published, 발행일, 게시일, 업로드일 |

### Optional Properties (auto-detected for enrichment)

| Purpose | Notion Type | Auto-match names |
|---------|-------------|------------------|
| Thumbnail | url | Thumbnail, 썸네일 |
| Summary | rich_text | Summary, 요약, AI요약 |
| Topics | multi_select or rich_text | Topics, 주제, 태그, Tags |
| ViewCount | number | Views, 조회수 |
| LikeCount | number | Likes, 좋아요, 좋아요수 |

### Notes

- **Title property as VideoID:** The title property stores the video ID (e.g., `dQw4w9WgXcQ`) and serves as the upsert lookup key. This prevents duplicates on re-runs. The human-readable video title goes into a separate rich_text property.
- **Type-first matching:** The title type is unique per database (always unambiguous). For url, number, and rich_text types, name patterns resolve ambiguity.
- **Ambiguity handling:** If multiple properties match a required purpose, present options and ask the user to choose.
- **Summary property:** If no Summary property is found, AI summaries will be displayed in the final report but not written to Notion.

## Workflow

This workflow runs fully automatically. For multiple channel IDs, Steps 1-6 repeat per channel. Steps 7-9 run once after all channels are processed.

### Step 1: Fetch RSS feed

```bash
curl -s "https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID"
```

Parse the XML response to extract `<entry>` elements. From each entry, collect:
- `<yt:videoId>` — video ID
- `<title>` — video title
- `<published>` — publish date (ISO 8601)
- `<link href="..."/>` — video URL
- `<media:group><media:thumbnail url="..." /></media:group>` — thumbnail URL

If the feed returns empty or an error, log the failure and skip to the next channel.

### Step 2: Filter by date

Filter entries to only those published within the last **N days** (default: 7).

If no entries remain, log "No videos published in the last N days for channel [ID]" and skip to the next channel.

### Step 3: Filter out Shorts

For each remaining video, check the video type:

```bash
mmk youtube videotype <video_id> -o json
```

Keep only entries where `type` is `"video"`. Discard entries where `type` is `"short"`.

If no videos remain, log "All recent videos are Shorts for channel [ID]" and skip to the next channel.

### Step 4: Get metadata

```bash
mmk youtube metadata <video_id> -o json
```

Collect from each response: title (authoritative), author/channel name, viewCount, likeCount, publishDate.

If metadata fails for a video, use the RSS title and channel name as fallback.

### Step 5: Get transcripts

```bash
mmk youtube transcript <video_id> -o json
```

Collect the `content` field. If a video has no captions, mark as "No transcript available" and continue.

### Step 6: AI-summarize

For each video:
1. If transcript available: generate a concise summary (2-4 sentences) covering the main topic and key points, plus 3-5 topic tags
2. If no transcript: generate a brief description from the title and metadata only, noting "(from metadata only)"

### Step 7: Discover Notion database schema

```bash
mmk notion database schema --database-id <db-uuid> --enhanced -o json
```

#### 7a. Required property discovery

Map database properties to required purposes using **type-first, then name-pattern** matching:

1. **VideoID:** Use the single `title` type property (always unambiguous).
2. **Title:** Find `rich_text` matching: `Title`, `제목`, `영상제목`.
3. **Channel:** Find `rich_text` matching: `Channel`, `채널`, `채널명`.
4. **URL:** Find `url` type properties. If exactly one, use it. If multiple, match by name: `URL`, `링크`, `영상링크`.
5. **Published:** Find `date` type properties. If exactly one, use it. If multiple, match by name: `Published`, `발행일`, `게시일`, `업로드일`.

If any required purpose has zero candidates, stop with a clear error listing what is missing and what types/names are expected.

#### 7b. Optional property discovery

Scan remaining properties for enrichment fields:

1. **Thumbnail:** `url` type matching `Thumbnail`, `썸네일`
2. **Summary:** `rich_text` type matching `Summary`, `요약`, `AI요약`
3. **Topics:** `multi_select` or `rich_text` type matching `Topics`, `주제`, `태그`, `Tags`
4. **ViewCount:** `number` type matching `Views`, `조회수`
5. **LikeCount:** `number` type matching `Likes`, `좋아요`, `좋아요수`

Report discovered mappings and continue automatically.

### Step 8: Upsert to Notion

Build the upsert data array using the discovered property names. Replace property keys with actual Notion property names from Step 7.

```bash
mmk notion database upsert --database-id <db-uuid> --lookup <title-property-name> --data '[
  {"<VideoID>":"abc123","<Title>":"How to Cook Pasta","<Channel>":"ChefChan","<URL>":"https://www.youtube.com/watch?v=abc123","<Published>":"2026-03-20","<Thumbnail>":"https://i.ytimg.com/vi/abc123/hqdefault.jpg","<Summary>":"A guide to making fresh pasta...","<ViewCount>":12345,"<LikeCount>":890},
  {"<VideoID>":"def456","<Title>":"Kitchen Tips","<Channel>":"ChefChan","<URL>":"https://www.youtube.com/watch?v=def456","<Published>":"2026-03-18","<Summary>":"Tips for organizing...","<ViewCount>":8901,"<LikeCount>":456}
]' -o json
```

Only include optional properties that were successfully discovered in Step 7b. For Topics with `multi_select` type, pass as an array (e.g., `["cooking","pasta"]`). For `rich_text` type, pass as a comma-separated string.

### Step 9: Present final summary

Display a single table with all results across all channels:

```
# YouTube Channel Summary Complete
- Channels processed: 3
- Period: Last 7 days
- Videos found: 15 | Shorts filtered: 4 | Regular videos: 11
- Transcripts: 9 available, 2 unavailable
- Upserted to Notion: 11 records (8 new, 3 updated)

| # | Channel  | Title              | Published  | Views  | Likes | Topics            | Summary                    |
|---|----------|--------------------|------------|--------|-------|-------------------|----------------------------|
| 1 | ChefChan | How to Cook Pasta  | 2026-03-20 | 12,345 | 890   | cooking, pasta    | A guide to making fresh... |
| 2 | ChefChan | Kitchen Tips       | 2026-03-18 | 8,901  | 456   | kitchen, tips     | Tips for organizing...     |
| 3 | DevTube  | Building CLI Tools | 2026-03-19 | 5,432  | 321   | CLI, Node.js      | Demonstrates building...   |
```

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `channel-id` | Yes | One or more YouTube channel IDs (comma-separated or listed individually) |
| `database-id` | Yes | Notion database ID or URL |
| `days` | No | Number of days to look back (default: 7) |

## Error Handling

| Scenario | Action |
|----------|--------|
| Auth failure on any mmk command | Stop workflow, report `mmk auth login` needed |
| Invalid channel ID or empty RSS feed | Log warning, skip to next channel |
| RSS fetch failure (curl error) | Log warning, skip to next channel |
| No videos in date range for a channel | Log, skip to next channel |
| All videos are Shorts for a channel | Log, skip to next channel |
| All channels yield zero videos | Report "No regular videos found across all channels" and stop |
| videotype command fails for a video | Skip that video, log warning, continue |
| metadata command fails for a video | Use RSS title/channel as fallback, continue |
| Transcript unavailable | Summarize from metadata only, continue |
| Database not found | Stop, report error with database-id |
| Required property not discovered | Stop, list missing purpose and expected types/names |
| Property discovery ambiguous | Present candidates, ask user to choose |
| Upsert partial failure | Log failed records, report partial results |
| No Summary property in database | Display summaries in output but skip writing them |

## Example Session

```
User: /mmk-youtube-channel-summary
      Channels: UC_x5XG1OV2P6uZZ5FSM9Ttw, UCddiUEpeqJcYeBxX1IVBKvQ
      Database: abc123-def456
      Last 7 days

Claude: [Channel 1/2] UC_x5XG1OV2P6uZZ5FSM9Ttw
       [Step 1] Fetching RSS feed... 15 entries found.
       [Step 2] Filtering to last 7 days... 10 videos.
       [Step 3] Checking video types... 8 regular, 2 Shorts filtered.
       [Step 4] Fetching metadata... 8/8 complete.
       [Step 5] Fetching transcripts... 7/8 available, 1 no captions.
       [Step 6] Generating summaries... 8/8 complete.

       [Channel 2/2] UCddiUEpeqJcYeBxX1IVBKvQ
       [Step 1] Fetching RSS feed... 12 entries found.
       [Step 2] Filtering to last 7 days... 6 videos.
       [Step 3] Checking video types... 5 regular, 1 Short filtered.
       [Step 4] Fetching metadata... 5/5 complete.
       [Step 5] Fetching transcripts... 5/5 available.
       [Step 6] Generating summaries... 5/5 complete.

       [Step 7] Discovering Notion database schema...

       Property mapping (auto-detected):
         VideoID   -> "VideoID" (title)
         Title     -> "제목" (rich_text)
         Channel   -> "채널" (rich_text)
         URL       -> "URL" (url)
         Published -> "게시일" (date)

       Optional fields discovered:
         Thumbnail  -> "썸네일" (url)
         Summary    -> "AI요약" (rich_text)
         Topics     -> "태그" (multi_select)
         ViewCount  -> "조회수" (number)
         LikeCount  -> "좋아요" (number)

       [Step 8] Upserting 13 videos to Notion...
       Upserted: 13 records (11 new, 2 updated).

       [Step 9] Complete!
       - Channels processed: 2
       - Period: Last 7 days
       - Videos: 16 found, 3 Shorts filtered, 13 regular
       - Transcripts: 12 available, 1 unavailable
       - Upserted: 13 records (11 new, 2 updated)

       | # | Channel    | Title                | Published  | Views  | Topics            | Summary                    |
       |---|------------|----------------------|------------|--------|-------------------|----------------------------|
       | 1 | ChefChan   | How to Cook Pasta    | 2026-03-20 | 12,345 | cooking, pasta    | A guide to making fresh... |
       | 2 | ChefChan   | Kitchen Organization | 2026-03-19 | 8,901  | kitchen, tips     | Tips for organizing...     |
       | 3 | DevTube    | Building CLI Tools   | 2026-03-19 | 5,432  | CLI, Node.js      | Demonstrates building...   |
       | ...                                                                                                       |
```

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-youtube](../mmk-youtube/SKILL.md) — All YouTube commands overview
- [mmk-youtube-videotype](../mmk-youtube-videotype/SKILL.md) — Video vs Short detection
- [mmk-youtube-metadata](../mmk-youtube-metadata/SKILL.md) — Video metadata
- [mmk-youtube-transcript](../mmk-youtube-transcript/SKILL.md) — Video transcript
- [mmk-notion-database](../mmk-notion-database/SKILL.md) — Schema, query, upsert commands
