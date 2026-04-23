---
name: mmk-plaud-search
description: Search Plaud transcripts and filenames. Triggers on "plaud search", "search recordings", "find recording", "search plaud transcripts", "plaud query".
allowed-tools: Bash(mmk *)
---

# MMK Plaud — Search

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-plaud`](../mmk-plaud/SKILL.md) — includes credential overrides and requirements.

Always use `-o json` when parsing results or composing with other commands.

---

## search — Search transcripts and filenames

```bash
mmk plaud search --query meeting -o json
mmk plaud search --query standup --date-from 1704067200000 --date-to 1735689599000 -o json
```

**Required:** `--query` (or `-q`) — the search query

**Optional:**

| Flag | Default | Description |
|------|---------|-------------|
| `--date-from` | | Earliest `start_time` in epoch milliseconds |
| `--date-to` | | Latest `start_time` in epoch milliseconds |

Use this for user-facing transcript search. For diagnostic access to the search history endpoint, see `mmk plaud admin search history` in [`mmk-plaud-admin`](../mmk-plaud-admin/SKILL.md).

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-plaud](../mmk-plaud/SKILL.md) — All Plaud commands overview, credential overrides
- [mmk-plaud-file](../mmk-plaud-file/SKILL.md) — List recordings and fetch transcripts/summaries
- [mmk-plaud-admin](../mmk-plaud-admin/SKILL.md) — Low-level `search history` diagnostics
