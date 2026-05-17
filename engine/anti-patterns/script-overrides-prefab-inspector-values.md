---
type: anti-pattern
project: timber-tycoon
suggested-category: engine/anti-patterns
tags: [unity, prefabs, inspector, scripts, debugging]
date: 2026-05-17
status: draft
---

# ANTI-PATTERN: Script Overwrites Prefab Inspector Values (DirtClump Case)

## The trap
Setting values in the Inspector on a prefab, then having code in `Awake` or `Start` overwrite those values with hardcoded constants. The designer edits the Inspector field and nothing changes in Play Mode. "I set it to 0.5 but in Play Mode it's 0.05."

## Why it fails
Code runs AFTER the Inspector-set value is loaded. The hardcoded override wins every time. The designer has no way to know from the Inspector that their edits are being silently discarded.

## Symptoms
- Inspector edit produces no in-game change
- Value in Play Mode differs from value in Inspector
- Designer spends 20+ minutes on each encounter until they find the code override

## Correct approach
Two fixes:

**Fix 1 (preferred when value is designer-tunable):** Remove the hardcoded override. Trust the Inspector value. The prefab IS the source of truth.

**Fix 2 (when value is mathematical/derived):** Read from a ScriptableObject or constant. Make it explicit that code controls this value — don't put it in an Inspector field that looks editable.

Detection: if Inspector edit produces no in-game change, `grep -r "fieldName" Assets/ --include="*.cs"` to find the override.

General rule: **NEVER overwrite Inspector-set values from code unless intentional and documented.**

See also: [[script-overrides-prefab-inspector]] for the more severe CapsuleCollider variant that causes physics failure.
