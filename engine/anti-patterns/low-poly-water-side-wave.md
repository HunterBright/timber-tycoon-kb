---
title: 'ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves'
type: anti-pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- shader
- water
- vertex-displacement
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves

## The trap
The intuitive first-pass water shader applies sinusoidal vertex displacement equally on X and Z axes. This seems correct - water should move. But the result looks wrong: the river "wobbles" perpendicular to its flow direction, like a trapped pond, not a flowing river.

## Why it fails
`displacement = sin(time + X*freq) + sin(time + Z*freq)` is symmetric in X/Z - it moves the water equally in all horizontal directions. A flowing river should have strong displacement along its flow axis and minimal displacement across it.

## Symptoms
- River surface oscillates sideways toward the banks
- Water looks like it's sloshing in a box, not flowing in one direction
- Current illusion completely breaks

## Correct approach
Anisotropic displacement aligned to flow direction:
- Main wave along flow axis only (e.g., X-axis if flow is +X)
- Side wave damping: `_SideWaveDamping = 0.1` (10% of main amplitude max)
- Organic noise scroll for variation: `_FlowNoiseScale = 3.0`

Expose in shader: `_WaveSpeed`, `_WaveHeight`, `_SideWaveDamping`, `_FlowDirection (Vector)`.

Full corrected shader: `Assets/Project/Shaders/LowPolyWater.shader`.

See also: [[river-mesh-semi-ellipse-cross-section]] for the geometry that pairs with this shader.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: vertex-displacement, water, shader
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, urp
- [[20260606-0615-meandering-river-flow-map-baked-tangent|Curved/meandering water flow via a baked flow map (arc-length V + per-vertex tangent)]] - wspolne: water, shader
- [[20260718-0800-particle-visibility-water-sorting|Czasteczki "dzialaja, ale ich nie widac" - trzy niezalezne przyczyny przy wodzie]] - wspolne: water, urp
- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: shader, urp
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
