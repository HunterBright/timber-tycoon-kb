---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, isaveable, lifecycle, awake, start, save-system]
date: 2026-05-17
status: draft
---

# Awake-Init for ISaveable with Dependencies

## When to use
Any ISaveable component that depends on other components or services being ready at load time. If your ISaveable's `LoadSaveData` depends on state that's set up in `Start()`, the load will overwrite your initialized values.

## Steps
1. Move ALL dependency wiring to `Awake`:
   - `Services.Register<T>(this)` — register in ServiceLocator
   - sibling component caching (`rb = GetComponent<Rigidbody>()`)
   - prefab reference caching
2. Leave `Start()` for game logic that runs after scene is fully loaded
3. SaveManager.Start iterates all ISaveables and calls `LoadSaveData` — by this time, all Awakes have run

Lifecycle order:
```
Awake (ALL instances) → OnEnable → Start (ALL instances) → first Update
```

SaveManager.Start fires during the Start phase — after all Awakes, but Start order is undefined between objects.

## Why this works
`Awake` is called before `Start` for ALL instances in the scene. By wiring dependencies in `Awake`, you guarantee that when SaveManager's `Start` dispatches `LoadSaveData`, your component is fully initialized and ready to receive restored state.

## Trade-offs
None. This is strictly better than Start-init for ISaveable components. The only gotcha: don't call `Services.Get<T>()` in Awake unless you know T has already Awake-registered itself (registration must precede lookup).

## Variants
Same timing issue applies to: components that subscribe to GameEventSOs (subscribe in Awake/OnEnable), components that reference ServiceLocator services (get in Awake after registration).

See also: [[isaveable-contract]], [[race-condition-start-vs-instantiate-parameter]]
