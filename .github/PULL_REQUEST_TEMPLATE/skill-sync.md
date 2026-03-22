## Summary
- Sync skill docs with mmk CLI **v{version}** (commit `{commit}`)
- {change_summary}

## Skill Sync Report

### New Services
{new_services}

### New Sub-commands
{new_subcommands}

### New Commands
{new_commands}

### Changed Flags
{changed_flags}

### Removed Commands (MANUAL REVIEW)
{removed_commands}

## Files Changed
{files_changed}

## Test plan
- [ ] Verify new commands: `mmk {command} --help` matches documented flags
- [ ] Verify link integrity: all `../mmk-*/SKILL.md` paths resolve
- [ ] Verify command counts consistent across root, shared, and README
- [ ] Verify frontmatter: name, description, allowed-tools present
