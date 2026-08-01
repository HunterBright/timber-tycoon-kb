---
title: 'Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-18'
project: Kerf - Sawmill Tycoon
tags:
- blender
- ui-sprite
- bake
- orthographic-camera
- headless-render
- 9-slice
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2

## Problem
When baking a flat 2D UI sprite (e.g. a wooden plank button) with a headless
orthographic render, the first attempts left a transparent margin around the
shape - the geometry did not fill the target PNG frame edge-to-edge, breaking
9-slice (transparent margin → borders don't tile cleanly).

## Root cause
In Blender's orthographic camera, the rendered frame's WORLD width equals
`ortho_scale × 2`, NOT `ortho_scale`. So a plane of width `W` centred at the
camera fills the frame only when `camera.data.ortho_scale = W / 2` (for a
landscape frame where width is the larger sensor dimension). Setting
`ortho_scale = W` leaves the plane occupying half the frame → transparent border.

## Fix / rule
For a flat sprite of world-size `W` (long axis) to bleed edge-to-edge in an
ortho bake: `ortho_scale = W / 2`. Verify after render by checking that all four
edge-midpoint pixels are opaque (alpha 255) and the opaque bounding box spans the
full PNG dimensions (X 0..width-1, Y 0..height-1).

## Context
- Headless Blender 5.1, `blender --background --python`, EEVEE Next, transparent film.
- Output: 820×256 RGBA PNG, light-oak plank, straight edges, horizontal grain;
  consumed by Unity as a horizontal 9-slice button background (border 120/0/120/0).
- Project gotcha that recurs across UI bakes - promote if it saves re-deriving the
  ×2 framing each time a flat sprite is baked.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260615-2045-blender-voronoi-round-knots|Proceduralne okrągłe sęki w Blenderze: Voronoi F1, nie DISTANCE_TO_EDGE (+ kompensacja proporcji)]] - wspolne: ui-sprite, blender
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] - wspolne: bake, blender
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] - wspolne: bake, blender
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: bake, blender
- [[20260612-1845-blender-9slice-ui-sprites|Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)]] - wspolne: 9-slice, blender
<!-- /POWIAZANE:auto -->
