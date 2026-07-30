---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, raycast, interaction, layermask, physics, fpp]
severity: medium
time_lost: ""
date: 2026-07-07
status: draft
applies_to: [unity]
---

# Single masked Physics.Raycast for "look-at to interact" gets eaten by a non-interactable collider on a masked layer

## Problem
FPP "press E to use" system: player parks a car on a loading ramp, then cannot enter the car. The interaction prompt only appears when the car is NOT overlapping the ramp. Any interactable placed behind a thin, non-interactable obstacle (ramp lip, low wall, prop) at close range becomes unusable even though the interactable's own trigger is right there.

## Root cause
The look-at detector cast a single `Physics.Raycast(ray, out hit, range, interactableLayer)` and then looked for an `Interactable` on `hit.collider`. `Raycast` returns only the FIRST hit along the ray. If a collider that sits on a layer INCLUDED in the mask (here the ramp, on Default) is closer than the interactable's trigger, the ray stops on it, finds no `Interactable`, and the detector clears the current target. The interactable behind it is never considered. This is not a mask bug per se — it is "first hit wins" colliding with "the nearest thing on my mask has no Interactable".

## Solution
Replace the single ray with an all-hits query and pick the NEAREST hit that actually resolves to an interactable, skipping non-interactable colliders instead of stopping on them:
- `Physics.RaycastNonAlloc(ray, buffer, range, mask)` into a reusable `RaycastHit[]` (zero per-frame GC).
- Loop all hits, `GetComponentInParent<Interactable>()`; ignore hits with none; track the one with smallest `.distance`.
- Use that as the current target; clear only if no hit had an interactable.
This fixes every "obstacle in front of interactable" case generically, with no physics/layer/scene changes and no risk to load-bearing colliders (contrast: gating the obstacle's collider on/off can drop a car resting on it).

## What didn't work
- Considered toggling the obstacle collider's `enabled` based on a zone trigger — rejected because the ramp collider was potentially the car's parking surface (disabling it while the player is on foot could drop the car).
- Editing the interaction LayerMask to exclude the obstacle's layer works only if the obstacle sits on its own dedicated layer; fragile and can regress intended line-of-sight blocking.

## Transferability
Any raycast-driven interaction/aim/use system in any genre (FPP, TPP, point-and-click). The "nearest hit on my mask may not be the thing I want; scan all hits for the component" rule generalizes to hover-highlight, gaze selection, and world-space UI raycasting. Short range (≈3 m) makes "reach through a thin obstacle" acceptable; for long-range LOS you'd instead stop at the nearest solid blocker OR the nearest interactable, whichever is closer.

## Related
- [[20260614-1226-modal-ui-over-world-interactable-guard]]
- [[20260625-1841-unity-collision-for-gpu-instanced-props]]
