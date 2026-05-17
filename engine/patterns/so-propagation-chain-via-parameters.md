---
name: so-propagation-chain-via-parameters
description: SO references propagate via explicit parameter passing through spawn chains (ChoppableTree → Stump → GrowingTree), not singleton lookup
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, scriptableobject, propagation, parameter-passing]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# SO Propagation Chain via Parameter Passing

## When to use
Any spawn chain where child objects need to know the "type" context of their parent — tree species, item variant, mission context. Without explicit propagation, children must look up global state or receive no context.

## Steps

**Full tree lifecycle propagation chain in Timber Tycoon:**

```
ChoppableTree (TreeTypeData SO) 
    → spawns DiggableStump → passes treeTypeData explicitly
DiggableStump (receives treeTypeData)
    → spawns CollectableLog → passes treeTypeData explicitly
CollectableLog (carries treeTypeData = species)
    → player pickups, delivers to storage (species-aware)

DiggableStump (dug up)
    → spawns PlantingSpot (universal, no treeTypeData — accepts any sapling)
PickupSapling (carries treeTypeData of its species)
    → player plants in PlantingSpot → PlantingSpot reads from sapling
PlantingSpot (receives treeTypeData from sapling)
    → spawns GrowingTree → passes treeTypeData explicitly
GrowingTree (carries treeTypeData)
    → matures → replaces with ChoppableTree → passes treeTypeData
```

**Implementation — explicit Init() after Instantiate:**
```csharp
// ChoppableTree.SpawnStump()
var stump = Instantiate(treeTypeData.stumpPrefab, position, rotation);
stump.GetComponent<DiggableStump>().Init(treeTypeData);

// DiggableStump.SpawnLog()
var log = Instantiate(treeTypeData.logPrefab, position, rotation);
log.GetComponent<CollectableLog>().Init(treeTypeData);
```

**Why parameters, not singletons:** singletons = global state = "which tree was cut last?" ambiguity when 3 trees cut simultaneously. Explicit parameter = each spawned object owns its data, no ambiguity.

**Race condition prevention:** always use `Init()` immediately after `Instantiate`, before the first `Update()` tick (see [[race-condition-start-vs-instantiate-parameter]]).

## Why this works
Data flows forward through the spawn chain as explicit values. Each component is self-contained. 10 trees cut simultaneously = 10 parallel chains, zero interference.

## Trade-offs
- Every spawning component must know to call `Init()` — easy to forget on new spawn sites. Document or enforce via constructor injection
- PlantingSpot breaks the chain (universal, accepts any species via sapling) — this is intentional design: player choice of sapling = player choice of species
- Chain can become long (5+ steps) — document it as a diagram to prevent confusion when debugging

## Variants
- **Context object:** bundle multiple SO refs into a single `SpawnContext` class, pass one object instead of N parameters — useful when context grows large
- **Event-driven:** each stage raises a typed GameEventSO with the context payload — more decoupled but harder to trace

See also: [[scriptable-object-runtime-injection]], [[race-condition-start-vs-instantiate-parameter]]
