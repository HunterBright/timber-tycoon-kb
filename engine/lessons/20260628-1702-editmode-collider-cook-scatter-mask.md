---
title: Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- editor-tooling
- physics
- meshcollider
- procedural-scatter
- raycast
- edit-mode
- occupancy-mask
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders

## Context
A Unity editor scatter tool placed decorative flora and avoided roads/water purely through physics queries - `Physics.Raycast` (reject if the down-ray hits a non-terrain collider) and `Physics.CheckSphere(pos, r, roadLayerMask)` (reject near road colliders). A decorative road built as a scene-embedded mesh (Catmull-Rom ribbon, no FBX asset) kept getting flora on top of it even after a MeshCollider on the correct layer was added.

## Root cause
**Non-convex MeshColliders backed by a runtime-generated or scene-embedded mesh do NOT cook their collision data in Edit Mode.** Cooking happens on Play/scene-load. So at editor time:
- `Physics.Raycast` passes straight through the road collider and hits the terrain below → the "is this point over a road?" test returns false.
- `Physics.CheckSphere` against that collider returns false too.

The road genuinely had a `MeshCollider` on the `Road` layer (verified: `layer==6`, `hasMeshCollider==true`), yet every physics-based exclusion silently missed it because the scatter runs in Edit Mode where the collider isn't cooked. The same gotcha bites any editor verification that measures against such colliders.

## Fix (the reliable pattern)
Bypass physics. Read the target mesh's geometry directly and rasterize it into a 2D occupancy mask:
1. `GameObject.Find` the object; `GetComponentInChildren<MeshFilter>().sharedMesh`.
2. Transform vertices by `mf.transform.localToWorldMatrix` → world XZ.
3. Allocate a `bool[]` grid (e.g. 1 m cells) over the mesh AABB + padding.
4. For each triangle, solid-rasterize (cell-center point-in-triangle test) → robust at any tessellation.
5. Dilate by `ceil(clearance / cell)` for an edge buffer.
6. O(1) lookup `mask[iz*W + ix]` replaces the CheckSphere/raycast test.

This is deterministic, works in Edit Mode, needs no collider at all, and self-adapts (re-reads the live mesh each run). Verified by an integrity pass counting placed instances inside the mask = 0.

## Related pattern - exclude curved/parametric features with their AUTHORING math, not bounding rects
Same tool excluded a meandering river and a noisy-outline lake with axis-aligned rectangles. A straight rect over a river that follows `Zc(x)=8−4·sin(...)` clips one bank and leaves the other bare; an oversized rect around a lake leaves a wide bare ring. Fix: port the exact equations that BUILT the water (river centerline + half-width profile; lake `INNER_R·EllipseRadius(θ)·ShoreFactor(θ)`) into the exclusion test, with a small shore margin. The mask then hugs the real waterline symmetrically. A plain ellipse is NOT enough when the real outline has per-angle noise - it under-covers bulges (objects float on water) and over-covers pinches (new bare ring). Reuse the source-of-truth math.

## Takeaways
- Editor-time tools must not assume physics colliders are queryable - non-convex runtime/embedded MeshColliders are uncooked in Edit Mode. Read mesh geometry directly for masks/tests.
- For exclusion around procedurally-built features, reuse the generator's own equations rather than approximating with rectangles.
- Always add an integrity/audit pass that re-checks the output against the mask and prints expected-zero counts - it turns "looks right" into proof.
