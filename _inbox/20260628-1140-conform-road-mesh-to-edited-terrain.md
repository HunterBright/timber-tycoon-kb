---
title: Conforming an existing road/decal mesh to terrain that was edited later
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- terrain
- road
- meshcollider
- raycast
- drape
- mesh-subdivision
- trigger
applies_to:
- unity
source: ''
severity: medium
time_lost: ~1h diagnosis (false-positive measurements)
suggested-category: engine/lessons
---

# Conforming an existing road/decal mesh to terrain that was edited later

## Problem
A gravel road laid as a thin "skin" over terrain developed see-through gaps (grass poking through the gravel) after the terrain underneath was reshaped in later sessions. Goal: make the road hug the current terrain again.

## Root cause
Two independent, non-obvious causes — plus one misleading measurement:

1. **Draping vertices is not enough when the mesh is coarse.** The road was a sparse ribbon (~1100 verts over ~230 m). Projecting each existing vertex straight down onto the terrain (set Y = terrainY + offset) put every *vertex* exactly on the surface, but the large flat triangles *between* vertices still cut through terrain bumps (poke) and bridged over dips (float) by up to several metres. Confirmed: AT-VERTEX gap was a perfect +0.04 everywhere, yet between-vertex sampling still showed ±2–5 m deviation.

2. **A trigger BoxCollider masked the real surface in downward raycast probes.** The road object had TWO colliders: a solid `MeshCollider` (the actual surface) AND a flat `BoxCollider` with `isTrigger = true` (a "vehicle is on road" detection slab, ~1 m thick, not terrain-following). `Physics.Raycast`/`RaycastAll` hit the trigger box by default (`Physics.queriesHitTriggers` is true), so every conformity measurement reported the flat slab's gap and never changed no matter what was done to the mesh — a constant false reading that looked like "my edit isn't applying."

3. **Non-convex MeshCollider with a runtime/embedded mesh does not re-cook for edit-mode queries reliably.** Even after reassigning `sharedMesh`, null→reassign, toggling `enabled`, `Physics.SyncTransforms()`, and a full scene save+reload, edit-mode raycasts against the solid MeshCollider were partial/absent. It cooks correctly on play/load; do not trust edit-mode collider raycasts as verification.

## Solution
1. **Subdivide the mesh first, then drape.** Uniform midpoint subdivision (split every triangle into 4, share edge-midpoints via an edge→index dictionary to stay watertight — no T-junctions) until max edge < ~1.2 m, interpolating vertex colors / UVs at midpoints. Then raycast each vertex onto the Terrain layer and set Y = hit.y + small offset (road +0.04, centerline +0.06). 1100 → ~11.8k verts; trivial for the vertex/draw budget.
2. **Verify on the RENDER mesh, not the collider.** Sample each triangle's centroid (world space) and compare to terrain — this reads the actual visible geometry and is collider/frame independent. Result here: 0 % poke, 0 % float, gap +0.03..0.06 everywhere.
3. **When you must probe a collider, ignore triggers:** `Physics.Raycast(..., QueryTriggerInteraction.Ignore)` to skip detection-trigger slabs and hit the solid surface.
4. Assign the new mesh to both `MeshFilter.sharedMesh` and `MeshCollider.sharedMesh`; on `EditorSceneManager.SaveScene` Unity embeds the runtime mesh into the scene (survives reload — verified). MeshCollider re-cooks from the embedded mesh on load/play.

## What didn't work
- Plain per-vertex drape with no subdivision (verts perfect, surface still poked).
- Forcing collider re-cook in edit mode (null→reassign, enabled toggle, SyncTransforms, scene reload) — collider raycasts stayed frozen; chasing this wasted time because the numbers were coming from the trigger box anyway.

## Transferability
Any engine where decorative surfaces (roads, paths, decals, water edges) are baked as a "skin" over terrain: if the terrain is later edited, the skin must be re-conformed, and a sparse skin needs *subdivision* not just vertex projection. The trigger-collider-masks-raycast trap and the edit-mode MeshCollider cook caveat are general Unity gotchas for anyone measuring surface conformity by raycast.

## When to stop patching and regenerate (process lesson)
Fixing the crude *junction* of two hand-made road ribbons by patching (overlay a fillet patch + trim the ends) took 5+ visual iterations and never looked clean — because splicing 3 pieces of different widths over wiggly hand-authored geometry always leaves shoulders/steps/width-jumps somewhere. The fix was to **regenerate the whole road segment as ONE uniform-width spline ribbon** (Catmull-Rom centerline extracted from the old route but smoothed, constant width, draped to terrain), deleting the old pieces. One mesh = no seams by construction.
- Heuristic: **if fitting a single seam/junction runs past ~3–4 iterations, stop patching and regenerate the whole segment.** Patching N hand-made pieces is O(seams) places to go wrong; one generated ribbon has zero internal seams.
- Caveat when extracting a centerline from an existing curvy road by axis-aligned slices: sampling too coarsely (e.g. every 30 m) **aliases** a high-frequency serpentine — you get a different (smoother) route. Fine for decorative roads where buildings are set back; sample densely if assets are glued to the road edge.

## Related
- Timber Tycoon project memory: project_road_redrape_2026-06-28, project_dirt_road_regen_2026-06-28
