---
name: legacy-code-conflict-after-refactor
description: After moving SpawnStump to TreeFactory, old ChoppableTree.SpawnStump still called from quest script — two paths diverge silently
metadata:
  type: anti-pattern
  project: timber-tycoon
  suggested-category: engine/anti-patterns
  tags: [unity, refactoring, legacy-code, conflict, technical-debt]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ANTI-PATTERN: Legacy Code Conflict After Refactor

## The Trap
You refactor `ChoppableTree.SpawnStump()` → `TreeFactory.CreateStump()`. New code builds and passes smoke tests. Six sprints later, a quest script still calls `ChoppableTree.SpawnStump()`. Both paths now diverge (TreeFactory received new features, ChoppableTree's version didn't), producing inconsistent stump behavior in that one quest. Hard to debug because "it worked before."

## Why It Fails
- Old API not deleted → no compile error when old callers exist
- Smoke tests test happy path, not "all code paths use the new implementation"
- Multiple refactors accumulate → eventually a caller of caller of caller still uses the old path
- Has happened in TT at least 3 times in different forms

## Correct Approach

**Delete old API immediately** (best option):
```csharp
// Just delete ChoppableTree.SpawnStump() entirely
// Compile error immediately shows all callers
// Fix all callers in the same commit
```

**If deletion isn't possible** (external callers, published API):
```csharp
// Shim that delegates to new implementation
[Obsolete("Use TreeFactory.CreateStump() instead")]
public void SpawnStump() => TreeFactory.Instance.CreateStump(transform, treeTypeData);
```
- Old callers still work (shim keeps them functional)
- New code uses new path
- Compiler warning on every old caller = visible reminder to migrate
- Remove shim only after `grep` confirms zero callers

**Before any refactor — find all callers:**
```bash
grep -r "SpawnStump" Assets/ --include="*.cs"
# Migrate all before merging
```

**Special risk: debug/test flags**  
After refactor, check for leftover `testMode`, `autoStart`, `debugSpawn` booleans. These can silently activate the old code path in Play Mode even if production code is migrated.

## Detection Pattern
- Symptom: "this worked before refactor" — exact signal that an old code path is still active somewhere
- Debug: add `Debug.Log("[LEGACY] ChoppableTree.SpawnStump called")` temporarily — if it fires, the old path is still in use

## See also
[[before-delete-legacy-class-checklist]], [[migration-pattern-rollback-safety]]
