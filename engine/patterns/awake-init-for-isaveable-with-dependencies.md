---
title: Awake-Init for ISaveable with Dependencies
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- isaveable
- lifecycle
- awake
- start
- save-system
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Awake-Init for ISaveable with Dependencies

## When to use
Any ISaveable component that depends on other components or services being ready at load time. If your ISaveable's `LoadSaveData` depends on state that's set up in `Start()`, the load will overwrite your initialized values.

## Steps
1. Move ALL dependency wiring to `Awake`:
   - `Services.Register<T>(this)` - register in ServiceLocator
   - sibling component caching (`rb = GetComponent<Rigidbody>()`)
   - prefab reference caching
2. Leave `Start()` for game logic that runs after scene is fully loaded
3. SaveManager.Start iterates all ISaveables and calls `LoadSaveData` - by this time, all Awakes have run

Lifecycle order:
```
Awake (ALL instances) → OnEnable → Start (ALL instances) → first Update
```

SaveManager.Start fires during the Start phase - after all Awakes, but Start order is undefined between objects.

## Why this works
`Awake` is called before `Start` for ALL instances in the scene. By wiring dependencies in `Awake`, you guarantee that when SaveManager's `Start` dispatches `LoadSaveData`, your component is fully initialized and ready to receive restored state.

## Trade-offs
None. This is strictly better than Start-init for ISaveable components. The only gotcha: don't call `Services.Get<T>()` in Awake unless you know T has already Awake-registered itself (registration must precede lookup).

## Variants
Same timing issue applies to: components that subscribe to GameEventSOs (subscribe in Awake/OnEnable), components that reference ServiceLocator services (get in Awake after registration).

See also: [[isaveable-contract]], [[race-condition-start-vs-instantiate-parameter]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[isaveable-contract|ISaveable Contract]] - wspolne: isaveable, save-system
- [[20260712-1820-save-migration-schema-version-gate|Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu]] - wspolne: isaveable, save-system
- [[20260622-1412-saveload-order-event-doublecount|Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu]] - wspolne: isaveable, save-system
- [[20260702-2200-save-system-missing-key-reset|Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)]] - wspolne: isaveable, save-system
- [[20260710-1952-save-key-name-path-hash-collision|Klucz zapisu z hasha ścieżki NAZW = kolizja przy duplikatach obiektów]] - wspolne: isaveable, save-system
<!-- /POWIAZANE:auto -->
