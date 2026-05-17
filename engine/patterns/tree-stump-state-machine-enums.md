---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, state-machine, enum, tree-cutting, gameplay]
date: 2026-05-17
status: draft
---

# TreeState + StumpState Enums State Machine

## When to use
Any multi-stage interactable object with discrete states and transitions — specifically the tree/stump life cycle. Use when gameplay logic needs to query "what state is this object in?" to gate interactions.

## Steps
Define two state enums:
```csharp
enum TreeState { Standing, Chopping, Fallen, Debranching, ReadyToCut, Cutting, Done }
enum StumpState { InGround, Digging, Ejected, Collected }
```

- `ChoppableTree.cs` holds current `TreeState`, exposes state-change events
- Child GameObjects enabled/disabled based on state (e.g., axe minigame UI only active during `Chopping`)
- Player interaction blocked via state check: `if (state != TreeState.Standing) return;`
- ISaveable persists current state — partial progress on a tree survives reload

Transitions triggered by: player interaction, minigame completion, time elapsed.

## Why this works
Explicit enum state replaces chains of boolean flags (`if branchesRemoved && !logsSpawned`). State is queryable, visible in the Inspector during debugging, and impossible to represent as inconsistent combination.

## Trade-offs
More upfront design (enumerate all states before coding). Pays off immediately after the second state is added — flag-based code would already be an unmaintainable mess.

## Variants
Same pattern for: machine states (Idle/Feeding/Processing/Ejecting), NPC states (Driving/Parking/Parked/Leaving), door states (Closed/Opening/Open/Closing).

See also: [[scriptable-object-runtime-injection]] for data-driven state variants per tree species.
