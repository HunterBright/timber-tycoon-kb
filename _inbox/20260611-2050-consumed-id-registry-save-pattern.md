---
title: Central consumed-ID registry for scene-object persistence
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-11'
project: Timber_Tycoon
tags:
- unity
- save-system
- persistence
- scene-objects
- registry
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Central consumed-ID registry for scene-object persistence

## Problem
Scene objects that the player consumes/destroys at runtime (collectibles, pickups, one-shot
interactables) resurrect after save → load, because the scene file always loads them in their
default state and nothing re-applies the "already consumed" fact.

## Why per-object ISaveable is the WRONG fix for consumables
A destroyed (or deactivated → OnDisable'd) object unregisters from the save system, so its key
silently **vanishes from the next save file**. On the following load there is no entry saying
"this object is gone" → it resurrects. The bug only appears after a *second* save/load cycle,
which makes it easy to miss in testing.

## Pattern
One central registry MonoBehaviour (ISaveable, auto-bootstrap via
`[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterSceneLoad)]` + lazy `GetOrCreate()`
fallback, since that attribute fires once per play session, not per scene load):

- Saves a `HashSet<string>` of **consumed object IDs** (ID = unique GameObject name is enough
  for hand-placed scene objects).
- Consumable objects register in `Start` (`RegisterObject(id, handler)`), call `MarkConsumed(id)`
  at the moment of consumption, and implement an interface like
  `IQuestWorldConsumable.ApplyConsumedState(bool consumed)`.
- On `LoadSaveData` the registry re-applies state to ALL registered handlers **in both
  directions**: consumed → hide, not-consumed → fully restore (scale, colliders, flags).
  Bidirectional matters because loads can be in-place (no scene reload) and the player may load
  an OLDER save mid-session.
- Late-register safety: if an object registers after the load already happened, apply its state
  immediately inside `RegisterObject`.
- Objects should `SetActive(false)` instead of `Destroy()` so the handler reference stays valid
  for potential restore.

## Result
Idempotent across arbitrary save/load cycles, no duplicates, exact per-object identity
("player took 1 of 2 tools — exactly that one stays gone"). Registry owns the state independent
of object lifetime — same reasoning as a spawned-object registry ([[worldspawnregistry-style]]
respawn list), but for *removal* instead of *creation*.

## Related gotcha fixed alongside
Registering with the save manager in `OnEnable` races the manager's `Awake` (undefined order) —
registration silently fails. Register in `Start` (all Awakes are guaranteed done).
