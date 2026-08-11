---
title: Terrain Skirt Against the See-Through Gap
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-29'
project: Kerf - Sawmill Tycoon
tags:
- blender
- modeling
- terrain
- skirt
- seethrough-gap
- low-poly
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Terrain Skirt Against the See-Through Gap

## Problem
A prop with a FLAT bottom at Z=0 (a planting-hole mound, a rock base, a building footprint) sits fine on flat ground but, placed on uneven/sloped terrain, shows a visible see-through gap UNDER its rim on the downhill side - you can see daylight beneath the prop where the terrain has dropped below Z=0. The game does not cut a real hole in the terrain, so the flat bottom just hovers over the dip.

## Root cause
A flat bottom edge at exactly Z=0 only meets ground that is also at exactly Z=0. The moment the terrain dips below the prop's footprint, the bottom edge is now above the ground surface, exposing the gap.

## Solution
Extend the prop's outer perimeter downward into a **"skirt"** that goes below Z=0 (here to ~Z=-0.25), so geometry always sinks into the terrain on any slope and there is never a gap to see through.

Key construction rules:
- Extrude the **outer boundary edge loop** down - the skirt is welded to the existing outer ring (shares those verts, no duplicates, no seam gap).
- **Extrude ALONG each outer face's own slope direction, NOT straight down on global -Z.** Two short bands work well: an upper band that continues each segment's existing outer-wall slope (so the transition at the seam is smooth, no kink), then a lower band tapering down to the target depth.
- **Clamp the horizontal radius** as the sloped walls spread outward, so the widest point stays inside the gameplay footprint (here clamped to 0.49, footprint radius 0.5). Final model: 91 v / 162 tri, max radius 0.4898, mound height 0.1494, skirt bottom -0.25.
- Skirt face normals must point **outward** (recalculate normals outside after extruding) - an inward skirt renders black / invisible and re-creates the gap.
- The center floor stays a flat face at Z=0 - only the perimeter gets the skirt; the bottom is left open (no cap, it's buried).

## What didn't work
**First attempt: extrude the outer loop straight DOWN along global -Z.** This produced a smooth vertical cylindrical "barrel" tube hanging off the bottom edge, with a hard kink where the organic sloped mound suddenly became a straight tube - it read as an obvious unnatural collar. The fix was to continue each face's slope direction instead, so the skirt looks like the mound's own surface flowing down into the ground.

## Distinction from the ZERO-floating mandate
This is the *opposite* of floating, not a violation of [[zero-floating-zero-flickering-mandate]]. The mandate forbids geometry hovering ABOVE a surface with a visible gap. A skirt deliberately sinks BELOW the ground plane so it's buried - a negative-Z minimum here is correct and intended, not an error. When validating, don't flag `minY < 0` as a bug for buried-skirt props; only the visible floor must stay at Z=0.

## Transferability
Any prop placed by code/designer onto non-flat terrain without a real terrain cut: planting spots, rocks, foundations, fence posts, signage bases. Cheaper and more robust than conforming the mesh to the heightmap.

## Related
- [[zero-floating-zero-flickering-mandate]]
- [[asset-origin-bottom-center-convention]]
- [[fbx-export-standard-settings-blender-to-unity]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[cylindric-beams-visual-contrast|Cylindric vs Rectangular Beams for Visual Contrast]] - wspolne: modeling, low-poly, blender
- [[20260702-1135-lowpoly-thin-planar-leaves-antipattern|Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly]] - wspolne: modeling, low-poly, blender
- [[20260711-1647-blender-prop-contact-interpenetrate-not-gap|Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)]] - wspolne: modeling, low-poly, blender
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] - wspolne: skirt, blender
- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] - wspolne: terrain, low-poly, blender
- [[zero-floating-zero-flickering-mandate|ZERO Floating / ZERO Flickering Mandate]] - wspolne: modeling, blender
<!-- /POWIAZANE:auto -->
