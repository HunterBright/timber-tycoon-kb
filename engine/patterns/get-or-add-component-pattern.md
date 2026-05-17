---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, components, extension-method, refactoring]
date: 2026-05-17
status: draft
---

# GetOrAddComponent Extension Method

## When to use
Any place you would write `AddComponent<T>()` in a setup script, spawn handler, or editor tool — especially if that code might run more than once (re-runs, respawns, parking respawn).

## Steps
Define once in `Assets/Project/Scripts/Extensions/GameObjectExtensions.cs`:
```csharp
public static T GetOrAddComponent<T>(this GameObject go) where T : Component {
    T comp = go.GetComponent<T>();
    return comp ?? go.AddComponent<T>();
}
```

Use everywhere instead of bare `AddComponent`:
```csharp
vehicle.GetOrAddComponent<NPCVehicle>();
```

## Why this works
Calling `AddComponent<T>()` twice creates two instances of the same component. Both run Awake/Start. The second one wins at runtime; the first one is an orphaned instance that wastes memory and can cause subtle bugs. `GetOrAddComponent` makes the operation idempotent.

## Trade-offs
Tiny overhead: one extra `GetComponent` call. Always worth it for any setup code that might run multiple times.

## Variants
Same principle as `GetOrCreateChild(name)`, `GetOrAddLayer(tag)` — any "ensure X exists, create if missing" pattern.
