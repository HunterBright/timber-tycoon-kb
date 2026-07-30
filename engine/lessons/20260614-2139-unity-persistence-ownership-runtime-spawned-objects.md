---
title: Persistence of a runtime-spawned object must be owned by its longest-living relative, never by a transient sibling
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-load
- isaveable
- runtime-spawn
- object-lifetime
- persistence-ownership
applies_to:
- unity-save-systems
source: ''
severity: high
time_lost: ~3 sessions of iteration
promoted: '2026-07-30'
---

# Persistence of a runtime-spawned object must be owned by its longest-living relative, never by a transient sibling

## Problem
A dug tree-stump spawns a "planting hole" (PlantingSpot) at the tree base. The hole
persisted across save/reload - UNTIL the player loaded the dug stump onto a vehicle.
Then on reload the hole vanished. Symptoms cascaded: first a missing object, then a
DUPLICATE phantom object, then a "works only if you don't touch X" partial fix. Took
three plan/implement/test cycles to reach a stable design.

## Root cause
The hole had no save record of its own. Its persistence was piggy-backed on the
**stump** component: the stump's ISaveable blob stored "hole active + position" and the
stump's LoadSaveData re-spawned the hole. But the stump is a *transient* object - it is
`Destroy()`-ed the moment it's loaded onto the vehicle (becomes cargo data). When the
persister dies, the thing it was persisting dies with it on the next reload. A second
latent bug fell out of the same mistake: the hole's completion callback pointed at the
(now destroyed) stump - a dangling delegate.

## Solution
Move ownership to the **longest-living related object**. Here that's `ChoppableTree`
(a scene object that outlives every runtime stump/hole and already holds the prefab +
base position). The tree now: persists `holeActive + position` in its own save blob;
spawns/respawns the hole after its scene-reconstruct, independent of the stump; receives
the completion callback (so it survives the stump's death); and the stump merely
*delegates* hole creation to `parentTree`. The transient object stops persisting it.

Companion trick used for the related "infinite duplicate" bug: detect "this object was
consumed/removed" by checking `runtimeRef == null` (Unity fake-null) at save time - a
destroyed object reads null, so a spawner can record "don't re-spawn me" without any
explicit teardown hook.

## What didn't work
- Letting the transient sibling persist the child (the original design) - breaks the
  instant the sibling is destroyed.
- Dual ownership (sibling handles case A, parent handles case B) - the completion
  callback can't cleanly route to both, and you get double-spawns or stale flags.
- Relying on Unity's deferred/"late-register" load hook to re-apply state to a
  runtime-spawned ISaveable: `lastLoadedData` is null during the loader's main pass, and
  the object's Start() runs a frame later - fragile, timing-dependent, and was a no-op in
  the common in-place-load path. Synchronous/owner-driven restore is deterministic.

## Transferability
Genre-independent Unity save-system rule: for any object that is *spawned at runtime by
another object* (loot, holes, effects, attachments, child entities), the save authority
must be the longest-living relative or a central spawn registry - never a peer/child that
can be destroyed, consumed, or picked up mid-session. Test matrix for any such object:
reload after it is (a) left in place, (b) carried, (c) consumed/destroyed, (d) converted
into a different object. Each path must give exactly one correct result.

## Related
- [[project_tree_save_architecture]] (project memory - the 3-system tree persistence map)
