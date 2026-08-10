---
title: Zero-thickness surfaces make voxel remesh amputate parts of a model
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- blender
- remesh
- voxel
- solidify
- generative-3d
- trellis
- mesh-repair
applies_to:
- blender-5.x
- any-photogrammetry-or-ai-mesh-cleanup
source: ''
severity: high
time_lost: ~4 h across two sessions
promoted: '2026-07-30'
---

# Zero-thickness surfaces make voxel remesh amputate parts of a model

## Problem
An AI-generated character mesh (TRELLIS output, 194k tris) fell apart during cleanup:
after a 3 mm voxel remesh the body split into three loose pieces (torso 0-1.19 m,
head 1.18-1.78 m, vest 0.97-1.17 m), the head was discarded as the second-largest
"speck", and the measured height dropped from 1.78 m to 1.18 m. Visually the character
read as "missing a neck" and you could see inside the skull through the mouth.

Every naive fix made it worse: hole filling only got boundary edges from 12817 down to
9688, and morphological closing (displace +d, remesh, displace -d) amplified the split
because the erosion step ate the already-paper-thin neck.

## Root cause
The generator emits some clothing and neck regions as **single-sided surfaces with no
volume** - not thin solids, literally zero thickness. A voxel/level-set remesh converts
a *volume* to a new surface; a zero-thickness sheet encloses no volume, so it either
vanishes or survives only as sub-voxel debris. Wherever such a sheet was the only bridge
between two body parts (the neck), the parts became disconnected.

## Solution
Insert a SOLIDIFY step between hole filling and remeshing:

1. `holes_fill` + `triangulate` + `recalc_face_normals` (normals matter: solidify offsets
   along them).
2. Solidify, `offset = -1.0` (new layer goes inward, silhouette unchanged),
   thickness **greater than the voxel size**. Measured on a 3 mm voxel grid:
   - 4.0 mm -> sheets partly dissolve again, holes and non-manifold edges come back
   - 4.5-5.0 mm -> zero boundary edges, zero non-manifold edges, one connected piece
3. Voxel remesh, drop loose specks, decimate to budget.

Turn **"Even Thickness" (`use_even_offset`) OFF**. On this mesh it ran for over
10 minutes and got killed by a timeout; with it off the whole pipeline takes 26 s.
It changes nothing here because the added layer ends up inside a volume that is about
to be remeshed anyway.

## What didn't work
- Hole filling alone (9688 boundary edges survive; complex boundaries do not fill).
- Morphological closing by vertex displacement - erosion destroys thin parts.
- Old Remesh modes BLOCKS/SHARP as a second pass - same interior waste plus 38-133 new
  non-manifold edges.
- Decimate with a protected vertex group to keep detail on the head - protected vertices
  refuse to collapse at all, so the modifier cannot reach the target count
  (stuck at 350k instead of 12k). Raising the global budget is the predictable lever.

## Transferability
Any pipeline that cleans up generated or scanned meshes (TRELLIS, photogrammetry,
sculpt kitbashes) hits this: the input looks watertight in a viewport because backface
culling is off, and the defect only shows after volumetric processing. The diagnostic
that pinpointed it in one shot: count connected components *before* and *after* the
remesh and print each component's Z range. Three components with adjacent Z ranges means
a zero-thickness bridge, not a modelling mistake.

## Related
- trellis shell has no body underneath (if written)
- Project memory: project_worker_trellis_repair_2026-07-26

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260612-1530-fix-baked-solidify-wrong-direction|Fixing a baked-in Solidify applied in the wrong direction]] - wspolne: mesh-repair, solidify, blender
- [[20260809-2140-metakule-i-remesh-to-technika-bazowa-nie-wykonczeniowa|Metakule plus remesh wokselowy to technika BRYLY BAZOWEJ, nie wykonczeniowa - do postaci uzywaj loftu z funkcja ksztaltujaca]] - wspolne: remesh, blender
<!-- /POWIAZANE:auto -->
