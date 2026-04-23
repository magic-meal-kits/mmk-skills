---
name: mmk-plaud-doctor
description: Plaud connectivity check — validates bearer token and lists user + devices. Triggers on "plaud doctor", "plaud connectivity", "plaud health", "plaud check token", "is plaud working".
allowed-tools: Bash(mmk *)
---

# MMK Plaud — Doctor

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-plaud`](../mmk-plaud/SKILL.md) — includes credential overrides and requirements.

Always use `-o json` when parsing results or composing with other commands.

---

## doctor — Connectivity check: validates token + lists user and devices

```bash
mmk plaud doctor -o json
```

**Required:** none

Use this command first when setting up Plaud access or debugging auth issues — it confirms the configured bearer token is valid and returns the current user plus connected devices in one call.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-plaud](../mmk-plaud/SKILL.md) — All Plaud commands overview, credential overrides
- [mmk-plaud-user](../mmk-plaud-user/SKILL.md) — Show current Plaud user profile
- [mmk-plaud-admin](../mmk-plaud-admin/SKILL.md) — Low-level device list and feature access
