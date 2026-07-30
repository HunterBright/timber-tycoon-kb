---
type: pattern
project: Timber_Tycoon
suggested-category: engine/patterns
tags: [blender, solidify, mesh-repair, modeling, clearance]
date: 2026-06-12
status: draft
---

# Fixing a baked-in Solidify applied in the wrong direction

## Problem
A mesh was thickened with Solidify using the wrong offset sign (inward instead of outward), the modifier was applied, and the result exported. The generated inner shell intrudes into a functional volume (e.g. moving machine parts). No modifier left to flip — the error is baked into geometry.

## Pattern
The applied solidify result still contains the **original pre-solidify surface intact** — it's the shell on the side the offset started from (for inward solidify: the outer shell). So the correct result can be reconstructed exactly:

1. **Identify the two shells geometrically** (z-level rings + radial distance for cones; coordinate bands for box sections). Connected-component analysis is unnecessary if rims join them — classify per-vertex by position instead.
2. **Measure the original thickness** from shell-pair displacement: `t = |outer - inner| / cos(wall slope correction)` — for parallel shells offset along the wall normal, `t` is the perpendicular distance.
3. **Delete the generated shell's vertices** (bmesh, `context='VERTS'`) — this removes its faces and the rim faces with it, leaving the pristine original surface with original UVs.
4. **Re-solidify with the correct sign**: `offset=+1` keeps the original surface as the inner boundary and adds material along +normal (outward, if normals point outward — verify first, do NOT recalc normals on the open shell, orientation is already correct from the manifold source).
5. Apply, then recalc normals outside **after** (mesh is manifold again), verify: same vert/face counts as before, 0 non-manifold edges.

## Why it works
Solidify is reversible as long as the source shell survives. Topology and UVs come out identical to "what the artist should have done" — new shell faces inherit UVs from the source faces, same as the original wrong solidify did, so baked atlas textures keep working.

## Verification trick for clearance
Vertex-in-volume checks miss intersections when one mesh has long faces spanning the other's z-range (vertices only at the ends). Check per-vertex of the *moving* part against the analytic wall profile r(z) instead: `min over verts of (r_wall(v.z) - r_xy(v))` — positive everywhere = clears.
