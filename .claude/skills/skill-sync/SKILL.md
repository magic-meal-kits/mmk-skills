---
name: skill-sync
description: Detect and document CLI changes after a new mmk version. Crawls --help, diffs against SKILL.md files, applies updates. Triggers on "sync skills", "update skill docs", "new cli version", "skill sync", "cli changes".
user-invocable: true
allowed-tools: Bash(mmk *), Read, Edit, Write, Glob, Grep
---

# Skill Sync — Detect & Apply CLI Changes

After a new `mmk` CLI version ships, run this skill to find what changed and update the SKILL.md files in this repo.

**Diff-only mode:** If the user says "just show the diff" or "dry run", stop after Phase 4.

---

## Phase 1: Version Check

Get the current CLI version:

```bash
mmk version -o json
```

Report the version to the user before continuing.

---

## Phase 2: Crawl CLI Command Tree

Build the full command tree by traversing `--help` at every level.

### Step 2a — Top-level services

```bash
mmk --help
```

Parse the output to extract service names (e.g., `notion`, `paymint`, `threads`, `youtube`, `auth`, `config`). Skip `auth`, `config`, `version`, and `completion` — they are not documented as skills.

### Step 2b — Sub-commands per service

For each service:

```bash
mmk {service} --help
```

Extract sub-command names (resource groups like `page`, `workspace`, `team`, etc.).

### Step 2c — Commands per sub-command

For each sub-command:

```bash
mmk {service} {sub} --help
```

Extract individual commands (e.g., `invite`, `revoke`, `list`).

### Step 2d — Flags per command

For each command:

```bash
mmk {service} {sub} {command} --help
```

Extract flags, their defaults, and descriptions. Also capture any usage examples in the help output.

Build a data structure like:

```
{service} -> {sub} -> {command} -> [flags, description, examples]
```

---

## Phase 3: Inventory Existing Skills

Read all existing SKILL.md files to build the documented command inventory.

### Step 3a — Find all skill files

Use Glob to find `mmk-*/SKILL.md` in the repo root.

### Step 3b — Parse each sub-skill

For each sub-command skill (`mmk-{service}-{sub}/SKILL.md`):
- Extract the `## {command}` sections
- Extract documented flags under each command
- Build a parallel data structure matching Phase 2's format

### Step 3c — Parse root skills

For each root skill (`mmk-{service}/SKILL.md`):
- Extract the Sub-Command Skills table
- Note the command counts listed

### Step 3d — Parse shared skill

Read `mmk-shared/SKILL.md` and extract the Command Reference section with counts.

### Step 3e — Parse README

Read `README.md` and extract the Skill Inventory table and Architecture tree with counts.

---

## Phase 4: Diff Report

Compare the CLI tree (Phase 2) against the skill inventory (Phase 3). Report changes in this format:

```
## Skill Sync Report — mmk v{version}

### New Services
| Service | Sub-commands | Commands |
|---------|-------------|----------|
(or "None")

### New Sub-commands
| Service | Sub-command | Commands |
|---------|------------|----------|
(or "None")

### New Commands
| Service | Sub-command | Command | Flags |
|---------|------------|---------|-------|
(or "None")

### Changed Flags
| Service | Sub-command | Command | Change |
|---------|------------|---------|--------|
(or "None")

### Removed Commands (MANUAL REVIEW REQUIRED)
| Service | Sub-command | Command | Current Location |
|---------|------------|---------|-----------------|
(or "None")

### Count Mismatches
| Location | Documented | Actual |
|----------|-----------|--------|
(or "All counts consistent")
```

**Important:** Never auto-delete removed commands. Flag them for the user's manual review.

If the user requested diff-only mode, present the report and stop here.

---

## Phase 5: Apply Changes

Follow the CLAUDE.md checklists precisely. Read `CLAUDE.md` first to get the exact conventions.

### Update order (dependencies flow downward):

1. **New service** (if any) — follow "Adding a New Service" checklist:
   - Create root folder `mmk-{service}/SKILL.md` using root template
   - Create sub-skill folders per granularity rules
   - Update `mmk-shared/SKILL.md` — add to Command Reference
   - Update `README.md` — inventory table, architecture tree, install section

2. **New sub-command** (if any) — follow "Adding a New Sub-Command Skill" checklist:
   - Create folder `mmk-{service}-{sub}/SKILL.md` using sub-command template
   - Update root skill — add row to Sub-Command Skills table, adjust count
   - Update `README.md` — inventory table, architecture tree, skill list
   - Update `mmk-shared/SKILL.md` — adjust count in Command Reference

3. **New command** (if any) — follow "Adding a New Command" checklist:
   - Add `## {command}` section to the correct `mmk-{service}-{sub}/SKILL.md`
   - Update root skill — adjust command count
   - Update `mmk-shared/SKILL.md` — adjust count
   - Update `README.md` — adjust count

4. **Changed flags** — update the flag documentation in the relevant sub-skill

5. **Removed commands** — present to user for manual decision. Do not auto-delete.

### Template references

When creating new files, use the templates defined in CLAUDE.md:
- Sub-Command Skill template for `mmk-{service}-{sub}/SKILL.md`
- Root Skill template for `mmk-{service}/SKILL.md`

Always include:
- Correct frontmatter (`name`, `description`, `allowed-tools`)
- `> **Prerequisite:**` line linking to `mmk-shared`
- `-o json` reminder
- See Also section with links to shared + parent + siblings

---

## Phase 6: Verify

Run the CLAUDE.md verification checklist:

### 6a — Link integrity

```bash
for f in mmk-*/SKILL.md; do
  dir=$(dirname "$f")
  grep -oP '\.\./mmk-[^)]*SKILL\.md' "$f" 2>/dev/null | while read link; do
    resolved=$(echo "$dir/$link" | sed 's|[^/]*/\.\./||g')
    [ ! -f "$resolved" ] && echo "BROKEN: $f -> $link"
  done
done
```

### 6b — Frontmatter check

Every `SKILL.md` must have `name`, `description`, and `allowed-tools` (except `mmk-shared`).

### 6c — Prerequisite check

Every skill except `mmk-shared` must have a `> **Prerequisite:**` line.

### 6d — Count consistency

Verify that command counts agree across:
- Root skill Sub-Command Skills tables
- `mmk-shared/SKILL.md` Command Reference
- `README.md` Skill Inventory table and Architecture tree

Report any failures and fix them.

---

## Example Session

```
User: /skill-sync
Agent: Checking CLI version...
       mmk v1.7.0

       Crawling CLI command tree...
       Found: 4 services, 12 sub-commands, 33 commands

       Reading existing skill inventory...
       Documented: 4 services, 12 sub-commands, 31 commands

       ## Skill Sync Report — mmk v1.7.0

       ### New Commands
       | Service | Sub-command | Command | Flags |
       |---------|------------|---------|-------|
       | notion  | database   | export  | --format (csv|json), --filter |
       | notion  | database   | import  | --file, --merge |

       ### Changed Flags
       (None)

       ### Removed Commands
       (None)

       Shall I apply these changes?

User: yes

Agent: [Updates mmk-notion-database/SKILL.md with new commands]
       [Updates mmk-notion/SKILL.md count: 20 -> 22]
       [Updates mmk-shared/SKILL.md count: 20 -> 22]
       [Updates README.md counts]

       Running verification...
       All checks passed.
```
