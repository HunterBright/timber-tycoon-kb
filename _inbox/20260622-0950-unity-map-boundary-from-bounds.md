---
title: Invisible map boundary from live Renderer.bounds + foot-only gap via IgnoreCollision
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-22'
project: Kerf - Sawmill Tycoon
tags:
- unity
- level-design
- colliders
- characize-controller
- rigidbody
- map-boundary
- physics
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Invisible map boundary from live Renderer.bounds + foot-only gap via IgnoreCollision

## When to use
- Open-world / sandbox map where the player (and/or vehicles) must be contained inside the playable area, but the terrain has no natural wall everywhere.
- You want a "simple straight wall" perimeter, not a silhouette-fit collider chain.
- You need one or more openings (secret, level exit) that let the on-foot player through but block a physics-driven vehicle.

## Steps
1. **Compute the footprint from the LIVE terrain, not from code constants.** Read the terrain object's `MeshRenderer.bounds` (already world-space, axis-aligned). This is immune to the map not being centered at origin and to stale size constants elsewhere in the project. Cross-check against `MeshFilter.sharedMesh.bounds` transformed by `localToWorldMatrix`; warn on divergence.
2. From `bounds` derive `minX/maxX/minZ/maxZ`, terrain low Y. Apply an `inset` (tunable) for fit.
3. Build a 4-segment rectangular ring of non-trigger `BoxCollider`s. Make each edge box overrun the corner by `thickness` so adjacent boxes overlap — no pinholes at corners.
4. Height ≈ way above any reachable point (player jump height is tiny vs. a 100m wall); thickness several metres so a fast Rigidbody vehicle cannot tunnel (a 1500kg body at 25 m/s moves <0.5m per physics step — 5m thickness is huge margin).
5. **Gap = split one edge box into two, and fill the gap span with a "vehicle stopper" box.** The stopper blocks the Rigidbody vehicle. A tiny runtime component on the stopper calls `Physics.IgnoreCollision(playerCharacterController, stopperCollider, true)` on Start so the on-foot player passes through it. (CharacterController IS a Collider subclass, so IgnoreCollision works on it.)
6. Persist as plain scene GameObjects under a dedicated root (e.g. `--- BOUNDARY ---`), built once by an idempotent Editor menu item. Keep all numbers in a ScriptableObject config.
7. Locate the gap automatically by raycasting from map centre toward the target-of-interest object and finding which edge it exits; drop a movable marker there so a designer can nudge + rebuild.

## Why this works
- `Renderer.bounds` is the single source of truth for where the map actually is; nothing else can drift.
- A CharacterController never tunnels regardless of speed; only Rigidbodies need thick walls + (optionally) continuous collision detection.
- `Physics.IgnoreCollision` cleanly separates "player passes / vehicle blocked" at the SAME spot — layer-matrix tricks fail because CharacterController ignores the layer collision matrix and collides with everything. IgnoreCollision is per-collider-pair and deterministic.
- IgnoreCollision is NOT serialized and resets on scene load — hence it must be (re)applied at runtime in Start, not baked.

## Trade-offs
- Rectangle (AABB) sits slightly inside/outside the visible irregular terrain edge in places — acceptable for an invisible wall; tune with `inset`. A silhouette-fit chain hugs the edge but is fragile and seam-prone (vehicle can wedge in seams).
- The foot-only gap needs one small runtime MonoBehaviour (the gate). Fully "no runtime code" is impossible if you want player-pass/vehicle-block at one opening.

## Variants
- No-gap full ring (fail-safe default if no gap marker/target found).
- Vehicle-also-allowed gap: just omit the stopper.
- Dedicated `Boundary` layer (auto-registered in TagManager) so NPC pathfinding/raycasts can mask the wall out.
