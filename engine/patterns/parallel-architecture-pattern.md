---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, architecture, service-locator, scriptableobject, isaveable, migration]
date: 2026-05-17
status: draft
---

# Parallel Architecture Pattern (Locator + Events + ISaveable + Singleton)

## When to use
Mid-to-large Unity projects where you want to improve architecture incrementally without a big-bang refactor. Also: any project starting fresh that wants a robust, testable architecture from day 1.

## Steps
4 systems coexist in parallel:

**ServiceLocator**: `Services.Register<T>(this)` in Awake, `Services.Get<T>()` anywhere. Replaces `FindObjectOfType`. Zero scene dependency, fast lookup.

**GameEventSO**: ScriptableObject event channels. `OnMoneyChanged.Raise(amount)` from EconomyManager, `listener.Register(callback)` in any subscriber. Zero direct references between systems.

**ISaveable**: Three-method contract (`GetSaveKey`, `GetSaveData`, `LoadSaveData`). Auto-registered in SaveManager.

**Singleton (legacy)**: `Instance` static property. Still works during migration. New code should prefer ServiceLocator.

Migration rule: NEW code uses ServiceLocator + GameEventSO. OLD code migrated "boy scout" style when touched. Singleton calls stay working until migrated.

## Why this works
Big-bang architecture refactor breaks builds for weeks. Parallel coexistence means: ship the demo, migrate feature by feature, zero disruption.

## Trade-offs
4 patterns to understand for new contributors. Document clearly in CLAUDE.md. The parallel period should be bounded (define a migration sprint).

## Variants
Simpler version (smaller projects): just ServiceLocator + ISaveable (skip GameEventSO for direct method calls). Add GameEventSO when teams grow or systems need decoupling.

See also: [[game-event-so-event-channel]], [[isaveable-contract]]
