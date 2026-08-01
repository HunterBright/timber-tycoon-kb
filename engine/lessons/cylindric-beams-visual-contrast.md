---
title: Cylindric vs Rectangular Beams for Visual Contrast
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- low-poly
- modeling
- architecture
- visual-design
applies_to:
- blender-pipelines
source: ''
severity: medium
suggested-category: engine/lessons
time_lost: ''
---

# Cylindric vs Rectangular Beams for Visual Contrast

## Problem
Rectangular beams against rectangular walls visually merge - no separation, no structural read. The building looks like one undifferentiated blob of rectangles.

## Root cause
When all primitive shapes in a building are rectangular (walls, beams, pillars), there's no visual hierarchy to guide the eye. The structural elements disappear into the background.

## Solution
Use cylindric beams (logs) for structural elements; rectangular planes for walls:
- Beams: cylinder meshes, 0.15×0.15 cross-section × N meters, rotated to lie horizontal
- Naming: `Beam_North_Log`, `Beam_East_Top_Log` (suffix `_Log` confirms cylinder)
- Pillars: also cylindric (same shape language) → beams + pillars feel connected; walls feel like infill
- Result: support structure (cylinders) + cladding (planes) = clear industrial wood aesthetic

The sawmill went from "generic shed" to "wooden sawmill" with 20 minutes of work.

## What didn't work
All-rectangular beams - walls and beams merge visually, no structural read.

## Transferability
Visual contrast principle applies to any architectural modeling: differentiate structural layers with different primitive shapes. Log cabin (round logs vs flat interior planks), industrial factory (circular pipes vs flat walls), medieval (round columns vs flat stone walls).

## Related
- [Architectural naming convention](../patterns/architectural-naming-convention.md)
- [Collider distribution rule](../patterns/collider-distribution-rule.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[terrain-skirt-against-seethrough-gap|Terrain Skirt Against the See-Through Gap]] - wspolne: modeling, low-poly, blender
- [[20260702-1135-lowpoly-thin-planar-leaves-antipattern|Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly]] - wspolne: modeling, low-poly, blender
- [[20260711-1647-blender-prop-contact-interpenetrate-not-gap|Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)]] - wspolne: modeling, low-poly, blender
- [[zero-floating-zero-flickering-mandate|ZERO Floating / ZERO Flickering Mandate]] - wspolne: modeling, blender
- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] - wspolne: low-poly, blender
- [[20260702-1130-blender-review-render-color-fidelity|Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
