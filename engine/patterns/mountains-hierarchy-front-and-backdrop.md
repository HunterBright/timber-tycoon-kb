---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, blender, mountains, backdrop, double-sided, low-poly]
date: 2026-05-17
status: draft
---

# Mountains Hierarchy — Front Ring + Backdrop Double-Sided

## When to use
Any low-poly open-world or enclosed map that needs distant mountains without a visible horizon cut-off. The player shouldn't be able to see "where the world ends."

## Steps

**Two-FBX split:**

| FBX | Verts | Height | Geometry | Cull |
|-----|-------|--------|----------|------|
| `Mountains.fbx` | 8.2k | 40–90m | Low-poly front ring | Single-sided |
| `Mountains_Backdrop.fbx` | 2100 | 100–151m | 48 segments × 4 radial rings | **Double-sided** |

**Overlap rule:** outer edge of front ring intersects inner edge of backdrop by **10m** — no visible seam from any player camera angle.

**Why double-sided backdrop:** camera at high elevation or inside a mountain pass occasionally sees the backdrop from behind. `Cull Off` in shader (or double-sided mesh in Blender) prevents holes.

**Material:** `Mat_Backdrop` uses `Custom/VertexColorLit` with `_Brightness = 0.5` — desaturated/distant look that reads as "far away."

**Inner mountains v2 (detail layer):** 30 individual mountains, 7 profile variants, `Custom/MountainLayered` shader with 5 altitude-based color bands. Placed in-scene by designer, not procedural.

## Why this works
Front ring provides detailed, camera-close mountains. Backdrop provides a large-scale visual enclosure that's cheap (2100 verts). The 10m overlap guarantees no gap regardless of render distance or player camera angle.

## Trade-offs
- Double-sided geometry: 2× polygon budget for backdrop — acceptable at 2100 verts total
- Backdrop isn't part of the playable terrain; if the player reaches map edge, they'd clip into it (design the map to prevent this)
- Height values hardcoded to TT's map scale — adjust for different map sizes

## Variants
- **Single FBX:** merge front + backdrop into one mesh if vertex count allows; lose independent material control
- **Sprite backdrop:** billboard mountain texture behind mesh ring (lower quality, faster)

See also: [[fbx-export-standard-settings-blender-to-unity]]
