---
name: java-kotlin-no-suppress-warnings
description: Fix Java/Kotlin/Spring warnings at source, never suppress. Use for JVM/Spring code, warnings, lint, or @Suppress.
---

# No warning suppression

## Rule

Never silence warnings with `@Suppress`, `@SuppressWarnings`, `@file:Suppress`, `//noinspection`, `@SuppressLint`, etc.

**Exception:** user explicitly allows suppression in the task.

If suppression is the only fix and not allowed — **stop** and report: which warning, why root fix failed in scope, need permission or requirement change.

Do not add suppress silently or by default.

## Instead

1. Fix cause: types, null-safety, unused symbols, deprecated API.
2. If false positive — restructure code so analyzer passes (no suppress).
3. If fix is out of scope — stop and ask.

## Before done

- [ ] No new `@Suppress*` / `//noinspection` / `@SuppressLint` in diff
- [ ] If suppress present — explicit user permission quoted in reply
