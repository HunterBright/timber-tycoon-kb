---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, scriptableobject, runtime, references]
date: 2026-05-17
status: draft
---

# ScriptableObject Runtime-Assigned References

## When to use
Multiple content variants share the same prefab and only differ in data (e.g., multiple tree species using the same ChoppableTree prefab). Prefab variants per species would bloat the project; Inspector-binding requires one prefab per variant.

## Steps
Parent spawner code pattern:
```csharp
GameObject stump = Instantiate(stumpPrefab);
DiggableStump ds = stump.GetComponent<DiggableStump>();
ds.treeTypeData = this.treeTypeData; // pass from parent
ds.Initialize(); // explicit init after assignment — prevents race condition
```

Component rule:
- Has individual prefab fields AND a SO field
- `Start()` checks SO: if not null, read prefab refs from SO (overrides Inspector)
- If SO null: use Inspector-set values (fallback for scene-placed instances)
- NEVER use singleton lookup to find SOs — pass explicitly through spawn chain

## Why this works
Single prefab + different SO data = multiple content variants without prefab explosion. Explicit parameter passing avoids global state and makes data flow traceable.

## Trade-offs
Requires `Initialize()` method on every spawned component (see [[race-condition-start-vs-instantiate-parameter]]). Slightly more spawn code per callsite.

## Variants
Same pattern for: WeaponData (weapon variants), ItemData (item types), VehicleData (vehicle configs), NPCData (NPC personality configs).

See also: [[scriptable-object-runtime-injection]], [[so-propagation-chain-via-parameters]]
