---
name: universal-cleanup-post-migration
description: After replacing debug shortcuts with real interactions, remove the debug code. Sprint-end checklist: grep for #if UNITY_EDITOR and // TODO debug, review each, remove if obsolete.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, cleanup, debug-code, migration, technical-debt]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects, claude-code-projects]
---

# Universal Cleanup Post-Migration

## Rule

After every migration sprint (replacing debug shortcuts with real interactions), run cleanup:

```bash
# Find debug-only code blocks
grep -rn "#if UNITY_EDITOR" Assets/Project/Scripts/

# Find explicitly marked debug items
grep -rn "// TODO debug" Assets/Project/Scripts/
grep -rn "// INTENTIONALLY LOW" Assets/Project/Scripts/
grep -rn "// TEMP" Assets/Project/Scripts/
```

Review each result. If the shortcut has been replaced by real interaction: delete it.

## Common pattern in TT

1. Early development: "U key opens upgrade shop" added for testing (inside `#if UNITY_EDITOR`)
2. Migration: real kiosk interaction implemented → upgrade shop now opens through kiosk
3. Cleanup (often skipped): `#if UNITY_EDITOR` U-key shortcut still exists
4. Result: both routes active simultaneously — can cause state confusion

## Anti-pattern examples

- Cheat keys left in production builds (players find them)
- Test data visible in real UI (debug console logs in final game)
- Dev menu accessible via hidden key combo
- `DebugCheatKeys.cs` still referenced in scene after moving to production build

## When to run cleanup

- End of each sprint, before PR merge
- Before any build sent to testers
- Before EA/release milestone

## Marker convention in TT

All temporary code uses one of these explicit markers:
- `#if UNITY_EDITOR` — editor-only, but review if real feature replaced it
- `// TODO debug` — remove before ship
- `// INTENTIONALLY LOW` — debug value, see [[intentionally-low-maxcapacity-test-racks]]
- `// TEMP` — temporary until [feature] is implemented

All markers must be searchable. Never add debug code without a marker.

See also: [[legacy-code-conflict-after-refactor]], [[intentionally-low-maxcapacity-test-racks]], [[before-delete-legacy-class-checklist]]
