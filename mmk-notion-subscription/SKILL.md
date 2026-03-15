---
name: mmk-notion-subscription
description: Show Notion workspace subscription details — tier, type, Stripe/RevenueCat data. Triggers on "subscription", "workspace plan", "billing tier", "subscription details".
allowed-tools: Bash(mmk *)
---

# MMK Notion — Subscription

> **Prerequisite:** Read [`../mmk-shared/SKILL.md`](../mmk-shared/SKILL.md) for auth, global flags, and error handling.
> Parent: [`mmk-notion`](../mmk-notion/SKILL.md)

Always use `-o json` when parsing results or composing with other commands.

---

## subscription — Show workspace subscription details

Show workspace subscription details (tier, type, Stripe/RevenueCat data).

```bash
mmk notion subscription -o json
```

No flags required.

---

## See Also

- [mmk-shared](../mmk-shared/SKILL.md) — Auth, global flags, error handling
- [mmk-notion](../mmk-notion/SKILL.md) — All Notion commands overview
