---
name: rotating-directional-light-black-terrain
description: Rotating Directional Light for day/night cycle — at dawn/dusk goes BLACK, at night lights from below. Use static overhead light + skybox-driven ambient instead.
metadata:
  type: anti-pattern
  project: timber-tycoon
  suggested-category: engine/anti-patterns
  tags: [unity, lighting, day-night, directional-light, ambient]
  severity: high
  time_lost: 2d (iterated on rotation-based system before switching approach)
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ANTI-PATTERN: Rotating Directional Light for Day/Night Cycle

## The Trap
First-pass day/night: rotate the Directional Light to follow the sun's arc across the sky. Simple concept, immediate implementation. Result:
- **Dawn/dusk:** light shines horizontally → extreme shadow angles → terrain nearly black
- **Night:** light points up from below ground → world lit from underneath → upside-down creepy look
- **Exact horizon:** NaN in shadow computation → flickering black artifacts on terrain faces

## Why It Fails
Directional Light affects all scene objects with its angle. At horizontal angles, almost all surface normals face away from the light → ambient occlusion dominates → scene goes black. At night angles (below horizon), the light acts as an underground point, producing "anti-natural" upward shadows with no analogous real-world result.

The assumption "rotate light = simulate sun" breaks because: the math of surface normals + diffuse lighting produces extreme results at these angles, and there's no atmosphere to soften the effect.

## Correct Approach

**Static overhead Directional Light + skybox-driven ambient (TT approach):**
- Directional Light stays at approximately overhead angle (e.g., ~45° elevation)
- Time of day is represented by: skybox color shift + ambient light color (not rotation)
- At night: light intensity scales down to 0.1–0.2, color shifts to cool blue
- Skybox shader draws sun/moon position decoratively (see [[procedural-skybox-sun-moon-trick]])
- Result: no black terrain at any time of day, consistent shadow direction

**4-phase weighted color blend** (see [[four-phase-weighted-smoothstep-day-night]]):
- Dawn / Day / Dusk / Night as color keyframes
- `smoothstep` weighting between phases
- Light intensity + color + ambient all driven by one `_TimeOfDay` float

**Light intensity scaling:**
```
Midday: intensity 1.0
Dawn/Dusk: intensity 0.5
Night: intensity 0.15
```
No rotation needed.

## When Rotation Is Acceptable
- High-fidelity realistic games where physically accurate sun positioning is required (e.g., sun casts realistic shadows on architecture)
- With a proper skybox that handles all edge cases (horizon glow, atmospheric scattering)
- With explicit clamping to prevent below-horizon angles

## See also
[[four-phase-weighted-smoothstep-day-night]], [[procedural-skybox-sun-moon-trick]]
