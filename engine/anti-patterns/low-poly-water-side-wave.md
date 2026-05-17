---
type: anti-pattern
project: timber-tycoon
suggested-category: engine/anti-patterns
tags: [unity, urp, shader, water, vertex-displacement]
date: 2026-05-17
status: draft
---

# ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves

## The trap
The intuitive first-pass water shader applies sinusoidal vertex displacement equally on X and Z axes. This seems correct — water should move. But the result looks wrong: the river "wobbles" perpendicular to its flow direction, like a trapped pond, not a flowing river.

## Why it fails
`displacement = sin(time + X*freq) + sin(time + Z*freq)` is symmetric in X/Z — it moves the water equally in all horizontal directions. A flowing river should have strong displacement along its flow axis and minimal displacement across it.

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
