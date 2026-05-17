---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, refactoring, migration, rollback, architecture]
date: 2026-05-17
status: draft
---

# Backend Migration Pattern with Rollback Safety

## When to use
Any refactor that moves state or logic from one system to another (e.g., `SawmillManager.CurrentReputation` → `ReputationManager`). Never big-bang: split into 2 sprints per migration, each independently committable and smoke-testable.

## Steps

**Concrete example: SawmillManager.CurrentReputation → ReputationManager**

**Sprint 4a (backend only — "the scary part"):**
1. Create `ReputationManager` singleton with ISaveable
2. Move data: reputation int, level lookup
3. Move state transitions: `OnSale` → increments reputation
4. Deprecate (don't delete): `SawmillManager.CurrentReputation` → mark `[Obsolete]`
5. Deprecation shim: `public int CurrentReputation => ReputationManager.Instance.Current` (old callers still work)
6. Smoke test: sell product → `ReputationManager.Current` increments, `SawmillManager.CurrentReputation` returns same value via shim
7. **Commit.** System works. Old callers work. Rollback possible.

**Sprint 4b (UI consumers — "the boring part"):**
1. Update HUD widget to read from `ReputationManager`
2. Update kiosk UI to read from `ReputationManager`
3. Remove `[Obsolete]` shim after `grep` confirms zero remaining callers
4. Smoke test: reputation HUD updates, kiosk shows correct level
5. **Commit.** Migration complete.

**API deprecation (4a):**
```csharp
[Obsolete("Use ReputationManager.Instance.Current instead")]
public int CurrentReputation => ReputationManager.Instance.Current;
```

**Before removing deprecated code** — must pass the before-delete checklist (see [[before-delete-legacy-class-checklist]]):
```bash
grep -r "CurrentReputation" Assets/ --include="*.cs"
# Must return 0 matches before deletion
```

## Why this works
Each sprint produces a working, committable state. If 4b introduces a bug, 4a remains a known-good commit. No big-bang = no "everything broke at once" debugging sessions.

## Trade-offs
- Shim maintenance: `[Obsolete]` shim must match new API's behavior exactly. If new API has different semantics (returns float instead of int), shim can't perfectly bridge — document the difference
- Two-sprint cost: takes longer than one big commit. Justified for any migration touching ISaveable (save format changes = user data at risk)
- Sprint 4a is "scary": it touches save data if reputation was previously saved in SawmillManager. Verify ISaveable key migration carefully (old key → reads as 0 → new key saves correctly)

## Variants
- **Feature flag migration:** wrap new code in `#if USE_NEW_REPUTATION` compile flag → test both paths in parallel → remove flag when confident
- **Single-sprint for small migrations:** if migrating a non-ISaveable, non-critical field (e.g., moving a visual variable), one sprint is fine

See also: [[before-delete-legacy-class-checklist]], [[isaveable-contract]], [[storage-migration-primary-plus-legacy-fallback]]
