# MMK Skills — Project Conventions

This repo contains Claude Code skills for the [Magic Meal Kits CLI](https://magicmealkits.com). Every skill follows a strict GWS-style flat folder hierarchy. **Never deviate from these conventions.**

## Folder Structure

```
skills/mmk-shared/SKILL.md              ← Background skill (auth, flags, errors)
skills/mmk-{service}/SKILL.md           ← Root skill (overview + sub-command table + shared guidance)
skills/mmk-{service}-{sub}/SKILL.md     ← Sub-command skill (focused command docs)
skills/mmk-{recipe}/SKILL.md            ← Recipe skill (multi-step workflow)
.claude-plugin/marketplace.json          ← Plugin marketplace catalog
.claude-plugin/plugin.json               ← Plugin manifest
README.md                               ← Inventory, architecture, install instructions
```

- **One folder per skill.** Each folder contains exactly one `SKILL.md`.
- **Flat hierarchy.** Sub-command folders are siblings of their root, not nested inside.
- **Naming:** `mmk-{service}-{sub}` — lowercase, hyphen-separated. Match the CLI hierarchy.

## Granularity Rules

When adding a new service or commands, decide the split level:

| Service size | Split level | Example |
|-------------|-------------|---------|
| Many commands grouped by resource | Split at **resource-group** level | `mmk-notion` → `mmk-notion-page`, `mmk-notion-team` |
| Flat list of commands | Split at **individual command** level | `mmk-paymint` → `mmk-paymint-send`, `mmk-paymint-cancel` |
| Single command | No sub-skill needed — put in root | (rare) |

## Templates

### Sub-Command Skill (`mmk-{service}-{sub}/SKILL.md`)

```markdown
---
name: mmk-{service}-{sub}
description: {Focused trigger phrases}. Triggers on "{phrase1}", "{phrase2}", ...
allowed-tools: Bash(mmk *)
---

# MMK {Service} — {Sub} Commands

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-{service}`](../mmk-{service}/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## {command} — {one-line description}

\```bash
mmk {service} {sub} {command} {flags} -o json
\```

**Required:** ...
**Optional:** ...

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-{service}](../mmk-{service}/SKILL.md) — All {Service} commands overview
- [mmk-{service}-{sibling}](../mmk-{service}-{sibling}/SKILL.md) — {sibling description}
```

### Root Skill (`mmk-{service}/SKILL.md`)

```markdown
---
name: mmk-{service}
description: {Broad trigger phrases covering all sub-commands}
allowed-tools: Bash(mmk *)
---

# MMK {Service}

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.

{One-line description}.

Always use `-o json` when parsing results or composing with other commands.

## Sub-Command Skills

| Sub-command | Description | Skill |
|-------------|-------------|-------|
| `{sub}` | {desc} ({N} commands) | [`mmk-{service}-{sub}`](../mmk-{service}-{sub}/SKILL.md) |

## {Shared Guidance} (if applicable)
{e.g., License Resolution, Pagination, Positional Arguments — content that applies to ALL sub-commands}

## Tips
{Service-level tips}
```

### Recipe Skill (`mmk-{recipe}/SKILL.md`)

Recipes compose multiple CLI tools. They must include:
- `disable-model-invocation: true` in frontmatter (side effects require manual invocation)
- `> **Prerequisite:**` linking to mmk-shared AND relevant sub-skills
- Step-by-step workflow with error handling
- Parameters table and example session

## Cross-Reference Rules

Every link uses relative paths (`../mmk-{target}/SKILL.md`). Follow this matrix:

| From | To | Where |
|------|----|-------|
| Sub-skill | `mmk-shared` | Prerequisite line |
| Sub-skill | Parent root | Prerequisite line + See Also |
| Sub-skill | 1–2 siblings | See Also |
| Root | Each sub-skill | Sub-Command Skills table |
| Root | `mmk-shared` | Prerequisite line |
| `mmk-shared` | Each root | Command Reference section |
| Recipe | `mmk-shared` + relevant sub-skills | Prerequisite line |

## Frontmatter Rules

Every `SKILL.md` must have:

| Field | Required | Notes |
|-------|----------|-------|
| `name` | All | Must match folder name |
| `description` | All | Include trigger phrases for discoverability |
| `allowed-tools` | All except `mmk-shared` | Usually `Bash(mmk *)` |
| `disable-model-invocation` | Recipes only | Set to `true` |

## Adding a New Command

When a new command is added to an existing service:

1. **Find the right sub-skill** — determine which `mmk-{service}-{sub}` it belongs to
2. **Add the command section** to that sub-skill's SKILL.md (follow the `## {command} — {desc}` pattern)
3. **Update the root skill** — adjust command count in the Sub-Command Skills table
4. **Update `mmk-shared`** — adjust command count in Command Reference if total changed
5. **Update `README.md`** — adjust command count in inventory table and architecture tree

## Adding a New Sub-Command Skill

When a new resource group or command needs its own folder:

1. **Create folder** `skills/mmk-{service}-{sub}/`
2. **Create `SKILL.md`** using the sub-command template above
3. **Update root skill** — add row to Sub-Command Skills table, adjust total count
4. **Update `README.md`** — add to inventory table, architecture tree, skill list, install examples
5. **Update `mmk-shared`** — adjust command count in Command Reference

## Adding a New Service

When adding an entirely new service (e.g., `mmk-naver`):

1. **Create root folder** `skills/mmk-{service}/SKILL.md` using root template
2. **Create sub-skill folders** per granularity rules above
3. **Update `mmk-shared`** — add entry to Command Reference section
4. **Update `README.md`** — add to inventory table, architecture tree, install section, usage examples
5. **Run verification** (see below)

## Verification Checklist

Run after every change:

1. **Link integrity** — every `../mmk-*/SKILL.md` path resolves to an existing file
2. **Content completeness** — every CLI command appears in exactly one sub-skill
3. **Frontmatter** — every SKILL.md has `name`, `description`, and `allowed-tools` (except mmk-shared)
4. **Prerequisite** — every skill except mmk-shared has `> **Prerequisite:**` line
5. **See Also** — every sub-skill has See Also with at least shared + parent
6. **Counts match** — command counts in root, shared, and README all agree

```bash
# Quick link check
for f in skills/*/SKILL.md; do
  dir=$(dirname "$f")
  grep -oP '\.\./mmk-[^)]*SKILL\.md' "$f" 2>/dev/null | while read link; do
    resolved=$(echo "$dir/$link" | sed 's|[^/]*/\.\./||g')
    [ ! -f "$resolved" ] && echo "BROKEN: $f -> $link"
  done
done
```

## Style Rules

- **`-o json` reminder** — every skill includes "Always use `-o json` when parsing results or composing with other commands."
- **Code blocks** — use `bash` language tag for all CLI examples
- **Flag tables** — use `| Flag | Default | Description |` format for optional flags
- **Required/Optional** — use `**Required:**` and `**Optional:**` bold labels
- **No emoji** in skill files
- **Descriptions in frontmatter** — must include "Triggers on" phrases for Claude Code discoverability
