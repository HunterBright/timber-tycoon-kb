---
name: intentionally-low-maxcapacity-test-racks
description: During development, set LOW maxCapacity (Plank=10, Bark=4) to hit "rack full" edge cases in 30 seconds. Mark with comment "INTENTIONALLY LOW for testing". Revert before shipping.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [unity, testing, iteration, capacity, debug]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Intentionally Low maxCapacity for Test Racks

## Rule

During development, use LOW maxCapacity values on test racks to hit edge cases quickly. Mark all low values with a comment so they don't ship.

```csharp
// INTENTIONALLY LOW for testing — production: Plank=50, Bark=30
LogRack.maxCapacity = 5;    // production: 60
PlankRack.maxCapacity = 4;  // production: 50
BarkRack.maxCapacity = 2;   // production: 30
```

## Why

**Testing with production values:**
- PlankRack capacity = 50 planks
- Making 50 planks = ~20 minutes of gameplay
- "rack full" edge case = 20 minutes per test run

**Testing with low values:**
- PlankRack capacity = 4 planks
- Making 4 planks = 30 seconds
- "rack full" edge case = instant

Iterate 40× faster.

## Applies to

- All StorageRack components during feature development
- Machine buffer sizes (machine holds 3 inputs before blocking)
- Any "wait until N" threshold that blocks on a full loop

## Pre-launch checklist item

Search for "INTENTIONALLY LOW" before any build:
```bash
grep -r "INTENTIONALLY LOW" Assets/Project/
```

Replace all found values with production values before tagging a release build.

## Variant: test mode flag

For systems where debug values need to coexist with production values:
```csharp
[SerializeField] bool debugMode = false;
public int MaxCapacity => debugMode ? 4 : 50;
```

But the simpler comment-based approach is usually sufficient and less code to maintain.

See also: [[storage-rack-family-system]], [[universal-cleanup-post-migration]]
