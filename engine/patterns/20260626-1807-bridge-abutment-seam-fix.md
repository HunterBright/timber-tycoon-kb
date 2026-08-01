---
title: 'Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-26'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- terrain
- heightfield
- road
- bridge
- conform
- bvh
- low-poly
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment

## Problem
A road meeting a fixed bridge deck floats above / sinks under the terrain at the abutment, and the bank doesn't transition smoothly into the deck. Common after a map reshape moves the river/banks but the road end was pinned to deck height instead of conforming to terrain.

## Why it happens
- A low-poly heightfield has vertices on a coarse grid (e.g. even-Z rows). The bridge seam usually falls **between** rows, so the interpolated terrain surface at the seam dips toward the next (lower) row - there is no vertex to hold it at deck height. Pinning the road end to deck height then leaves it floating over that dip.
- The two banks are often **asymmetric**: one side sits below the deck (needs FILL), the other above it (needs CUT). A "fill-only" assumption fixes one side and ignores the other (the "road sinks under terrain" side).

## Pattern
1. **Probe the real mesh first** (raycast the actual terrain along the road column) - never trust the assumed profile.
2. **Insert one edge loop exactly at the deck seam** (`bmesh.ops.bisect_plane` on a constant-axis plane), clipped to the road-width band. Span the cut out to where the X-taper weight reaches 0 so the loop's terminal verts coincide with the original surface → zero-height end T-junctions, crack-free.
3. **Set the seam loop to `deck_top − conformOffset`** so a road conformed at `+conformOffset` lands flush on the deck. Allow BOTH raise and cut (per side). Clamp every moved vert `≤ local deck_top` (no poke-through) and keep a water/riverbed keep-out guard.
4. Leave the **under-deck** rows untouched (the bank dropping to water there is the hidden abutment underside; raising it would poke through the deck bottom).
5. **Re-export terrain, THEN re-drape the road** onto the new mesh via a BVH raycast: `roadY = terrainRaycast(x,z) + conformOffset`. Remove any previously-forced end-cap heights. The seam end-cap now lands exactly on the deck.

## Result
Road sits at exactly `terrain + offset` everywhere (no float/sink), seam meets the deck within <0.1 m, water and deck untouched. In our case only ~25 terrain verts + 17 road verts moved.

## Gotchas
- Measure the live conform offset (we found +0.02 m, while old scripts hardcoded +0.08 - the +0.08 was itself a float source).
- Terrain MeshCollider must re-cook after the mesh changes (reimport, or toggle the collider) or NPC/footstep raycasts and the next BVH read stale geometry.
- Keep deck geometry FIXED; always fix terrain-to-deck and road-to-terrain, never move an approved deck.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260730-2350-layer-clearance-height-over-body|Luz między warstwami ubrań: mierz WYSOKOŚĆ NAD CIAŁEM, nie odległość do najbliższej ścianki]] - wspolne: bvh, heightfield, blender
- [[20260626-1808-probe-heightfield-before-terrain-edit|Probe the real heightfield before scripting terrain edits - assumed profiles drift]] - wspolne: heightfield, terrain, blender
- [[20260609-0830-conform-terrain-to-path-via-per-x-centerline-profile|Conform a mesh terrain to a path by sampling its centreline per-axis (not a fixed scan line, not a single plane)]] - wspolne: road, terrain
- [[flatten-terrain-under-road|Flatten Terrain Under Road (Smoothstep Blend)]] - wspolne: road, terrain
- [[terrain-skirt-against-seethrough-gap|Terrain Skirt Against the See-Through Gap]] - wspolne: terrain, low-poly, blender
- [[20260628-1140-conform-road-mesh-to-edited-terrain|Conforming an existing road/decal mesh to terrain that was edited later]] - wspolne: road, terrain
<!-- /POWIAZANE:auto -->
