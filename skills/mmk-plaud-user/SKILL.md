---
name: mmk-plaud-user
description: Plaud user account info — show the current Plaud user profile. Triggers on "plaud user", "plaud me", "plaud profile", "plaud account info", "who am i on plaud".
allowed-tools: Bash(mmk *)
---

# MMK Plaud — User

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-plaud`](../mmk-plaud/SKILL.md) — includes credential overrides and requirements.

Always use `-o json` when parsing results or composing with other commands.

---

## user me — Show the current Plaud user profile

```bash
mmk plaud user me -o json
```

**Required:** none

Returns the profile associated with the configured Plaud bearer token.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-plaud](../mmk-plaud/SKILL.md) — All Plaud commands overview, credential overrides
- [mmk-plaud-doctor](../mmk-plaud-doctor/SKILL.md) — Connectivity check (user + devices in one call)
- [mmk-plaud-admin](../mmk-plaud-admin/SKILL.md) — Low-level user feature-access diagnostics
