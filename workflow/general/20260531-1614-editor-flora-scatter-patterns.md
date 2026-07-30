---
title: Reproducible Editor Flora Scatter onto a Mesh Terrain
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- editor-tool
- scatter
- raycast
- terrain
- foliage
- procedural
- placement
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Reproducible Editor Flora Scatter onto a Mesh Terrain

Pattern for scattering decoration (flora, rocks, debris) over a **mesh** terrain (not Unity
Terrain) from an editor tool, with self-validation and a clean undo handle.

## Core placement loop (per candidate)
1. Random (x,z) inside an approved zone rectangle (fixed `System.Random(seed)` → reproducible).
2. **Cheap rejects first**: edge inset; hard exclusion rectangles.
3. `Physics.Raycast` straight down from well above (Y≈150), `QueryTriggerInteraction.Ignore`.
4. **Accept only if `hit.collider.gameObject == terrainGO`.** This one rule auto-rejects roads,
   buildings, machines, trees, racks - anything with a collider sitting on the terrain. Tag the
   reject reason by `hit.collider.gameObject.layer` (e.g. Road layer) for the report.
5. Reject steep ground: `hit.normal.y < 0.7` (handles mountain bases / cliffs for free).
6. Min-spacing via a **spatial hash** (`Dictionary<long,List<Vector2>>`, cell = spacing, check
   3×3 neighbor cells) - O(n), not O(n²). Use a wider spacing for sparse "accents" than for the
   dense "undergrowth" base layer.

## Seat ON the ground regardless of pivot (key trick)
Don't trust the asset's pivot. Place at `hit.point`, then read the instance's **combined
renderer world bounds** and lift by `hit.point.y - bounds.min.y` (minus a ~3 cm embed). Now the
mesh *bottom* rests on the ground whether the FBX origin is at the base or the center. Verify by
re-raycasting at each instance's XZ and comparing **bounds.min.y** (not pivot.y) to ground.

## Water with no solid collider
A water plane/river often has no solid collider → the down-ray punches through to the riverbed
and you'd plant flora underwater. **Hard-exclude water by rectangle**, never rely on the raycast.

## Hygiene
- Parent everything under one `FloraScatter_Root` → undo = delete that one GameObject. Re-runs
  clear it first (idempotent + reproducible via fixed seed).
- **Mandatory test batch (~5%) + self-validate before the full run**: report placed/rejected
  counts by reason, sample N instances for floating/sunk + in-exclusion, render a top-down PNG.
  Only proceed to full once clean.
- **Check the vert budget before committing.** Sum existing scene `MeshFilter.vertexCount`
  (excluding the scatter root), subtract from the project cap (here 2,000,000), and size density
  to the remainder. Extrapolate from the 5% test (×20). Heavy items (bushes ~2300 v) dominate -
  bias density toward cheap undergrowth and keep expensive accents sparse.
- No colliders on scattered instances → they never interfere with subsequent raycasts.
- Save the scene exactly once, at the very end, in Edit Mode (never in Play Mode); back up the
  `.unity` first.
