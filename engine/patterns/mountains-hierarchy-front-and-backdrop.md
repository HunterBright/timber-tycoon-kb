---
title: Mountains Hierarchy - Front Ring + Backdrop Double-Sided
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- mountains
- backdrop
- double-sided
- low-poly
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Mountains Hierarchy - Front Ring + Backdrop Double-Sided

## When to use
Any low-poly open-world or enclosed map that needs distant mountains without a visible horizon cut-off. The player shouldn't be able to see "where the world ends."

## Steps

**Two-FBX split:**

| FBX | Verts | Height | Geometry | Cull |
|-----|-------|--------|----------|------|
| `Mountains.fbx` | 8.2k | 40-90m | Low-poly front ring | Single-sided |
| `Mountains_Backdrop.fbx` | 2100 | 100-151m | 48 segments × 4 radial rings | **Double-sided** |

**Overlap rule:** outer edge of front ring intersects inner edge of backdrop by **10m** - no visible seam from any player camera angle.

**Why double-sided backdrop:** camera at high elevation or inside a mountain pass occasionally sees the backdrop from behind. `Cull Off` in shader (or double-sided mesh in Blender) prevents holes.

**Material:** `Mat_Backdrop` uses `Custom/VertexColorLit` with `_Brightness = 0.5` - desaturated/distant look that reads as "far away."

**Inner mountains v2 (detail layer):** 30 individual mountains, 7 profile variants, `Custom/MountainLayered` shader with 5 altitude-based color bands. Placed in-scene by designer, not procedural.

## Why this works
Front ring provides detailed, camera-close mountains. Backdrop provides a large-scale visual enclosure that's cheap (2100 verts). The 10m overlap guarantees no gap regardless of render distance or player camera angle.

## Trade-offs
- Double-sided geometry: 2× polygon budget for backdrop - acceptable at 2100 verts total
- Backdrop isn't part of the playable terrain; if the player reaches map edge, they'd clip into it (design the map to prevent this)
- Height values hardcoded to TT's map scale - adjust for different map sizes

## Variants
- **Single FBX:** merge front + backdrop into one mesh if vertex count allows; lose independent material control
- **Sprite backdrop:** billboard mountain texture behind mesh ring (lower quality, faster)

See also: [[fbx-export-standard-settings-blender-to-unity]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] - wspolne: low-poly, blender
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] - wspolne: low-poly, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: low-poly, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: low-poly, blender
- [[20260612-1845-blender-9slice-ui-sprites|Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)]] - wspolne: low-poly, blender
- [[20260702-1135-lowpoly-thin-planar-leaves-antipattern|Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
