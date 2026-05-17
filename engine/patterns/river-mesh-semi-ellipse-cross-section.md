---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, blender, mesh, river, water, terrain-blending]
date: 2026-05-17
status: draft
---

# River Mesh Semi-Elliptical Cross-Section

## When to use
Generating a river mesh procedurally (via Blender Python or Unity C#) that looks like actual flowing water in a riverbed — not a flat plane painted blue.

## Steps
Cross-section formula per vertex (9 verts across width per section):
```python
t = abs(distance_from_centerline / half_width)  # 0 at center, 1 at bank
z = bank_z - depth * (1 - math.sqrt(1 - t*t))  # convex from above
```

Key parameters:
- Width: 3m per section (semi-ellipse ~2.7m visible concavity)
- Depth at centerline: 2–5m (varies along course)
- Centerline tracking: max 6m change per 5m step (prevents discontinuity)
- Edge tucking: `bank_z = min(z_ellipse, terrain_z - 0.15)` → river edges sink below terrain, hides seam

TT river: 113 sections × 5m spacing, 1017 verts total, exported as `River.fbx`.

## Why this works
Semi-ellipse gives the visual impression of a carved riverbed. Edge tucking hides the seam between river mesh and terrain mesh — without it, a visible geometric edge breaks the illusion.

## Trade-offs
Requires procedural mesh generation script (Blender Python or runtime C#). Static hand-modeled river is an alternative for simpler projects.

## Variants
Same formula for: canals, moats, valley cross-sections, footpaths that are slightly sunken into terrain.

See also: [[low-poly-water-side-wave]] anti-pattern for the water shader.
