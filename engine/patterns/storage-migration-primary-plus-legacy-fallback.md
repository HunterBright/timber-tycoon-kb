---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, refactoring, migration, save-compat, technical-debt]
date: 2026-05-17
status: draft
---

# Storage Migration: Primary New + Legacy Fallback

## When to use
Migrating a storage or data system (e.g., WarehouseManager → StorageManager) across multiple sprint cycles when not all callsites can be migrated at once. Preserves old save file compatibility during migration.

## Steps
1. Implement new system fully
2. At each callsite: new system is PRIMARY path, legacy is FALLBACK:
```csharp
// Sprint C2 — migration to StorageManager
int removed = StorageManager.Instance.ConsumeOneLog(species);
if (removed == 0) {
    // TODO Sprint C5: remove fallback once all systems migrated
    removed = WarehouseManager.Instance?.ConsumeOneLog(species) ?? 0;
}
```
3. Mark fallback with `// TODO Sprint X:` — findable via grep
4. At load time: read old save keys, write to new keys, clear old keys
5. Remove fallback ONLY when `grep -r "WarehouseManager" Assets/` returns zero hits

## Why this works
Hard migration breaks old saves and requires all callsites to be perfect on day 1. Soft migration with fallback means partial migration = functional system throughout. Old saves migrate transparently on first load.

## Trade-offs
~50 lines of fallback code per system. Temporary technical debt. Must actually remove fallbacks (set a sprint deadline, use grep enforcement).

## Variants
Same pattern for: database schema migrations, API versioning (v1 fallback while v2 rolls out), config file format changes.

See also: [[before-delete-legacy-class-checklist]], [[reputation-backend-migration-rollback-safety]]
