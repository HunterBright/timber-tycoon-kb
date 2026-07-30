---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, terrain, water, lowpoly, vertex-color, shoreline]
date: 2026-06-28
status: draft
severity: medium
---

# Low-poly lake shore looks jagged (serrated) — submerge the rim + widen the water, don't densify

## Symptom
A smooth water mesh sitting on a low-poly terrain (e.g. 2 m grid) shows a **serrated / stair-stepped shoreline** from above, plus an inconsistent dirt "halo" where grass should meet water.

## Root cause
- The square terrain grid quantizes the curved waterline contour → terrain "teeth" poke up through the flat water plane (the contour where terrain crosses the water Y wanders ±2-3 m on a 2 m grid).
- The shore vertex-color splat (carve-depth → grass/dirt/rock) makes **dirt dominant exactly at the waterline**, so bare earth shows.

## Fix (low-risk — NO terrain topology change, so NO NavMesh / collider rebuild)
1. **Submerge the rim band**: lower the terrain's shaped target at/just inside the shoreline to slightly BELOW the water plane (e.g. water 0.40 → shore 0.18), holding a thin submerged shelf out to ~1.05× the shore radius. The terrain only emerges just OUTSIDE the water disc.
2. **Widen + refine the visible water mesh** ~5% outward and raise its ring resolution (e.g. 72 → 160 segments) so the smooth disc edge becomes the visible shoreline and the terrain teeth sit hidden underwater.
3. **Repaint the shore vertex-colors** in a thin band around the local water-line to consistent grass (+a hairline sand), rock = 0 — for BOTH lake and river (river water-line rises along its length, so band relative to the local water height, not a fixed Y).

## Anti-pattern (avoid)
Densifying / Laplacian-smoothing the terrain rim to fix the jaggedness: it changes terrain topology → forces NavMesh + collider rebuild and risks cracks/T-junctions — for a low-poly game that doesn't need a perfectly smooth land contour.

## Preserve
Wading/interaction colliders usually live on the ORIGINAL meshes (not the merged visible water mesh) — don't touch them. Freeze any bridge/inlet zones during the re-sculpt.
