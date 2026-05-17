---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, urp, shader, skybox, sun-moon, procedural]
date: 2026-05-17
status: draft
---

# Procedural Skybox Sun/Moon Trick

## When to use
Low-poly / stylized day/night cycle that needs both sun and moon visuals in the skybox without a second light source or separate moon texture.

## Steps
Skybox uniforms set per-frame by `DayNightCycle.cs`:
- `_TimeOfDay` (0–1)
- `_SunDirection` (Vector3)

Day phase: `_SunDirection` = actual sun direction  
Night phase: `_SunDirection = -moonDir` → shader draws "sun" disc where the moon actually is

Sun rendering:
```glsl
float sunDisc = smoothstep(thresh, 1.0, dot(viewDir, _SunDirection));
float halo = pow(dot(viewDir, _SunDirection), 32.0);
```

Moon: enabled only at night (faded by `_TimeOfDay`), same east-west arc as sun (Minecraft-style).

Stars: procedural grid hash (~3% cell density), sinusoidal twinkle, only above horizon and at night.

## Why this works
Sun and moon are mathematically symmetric — they travel the same arc, just at opposite times. Using `-moonDir` as `_SunDirection` at night reuses the sun disc shader code for free moon rendering.

## Trade-offs
Moon and sun always travel the same arc (stylized, not realistic). For realistic astronomical positioning, a second shader input is needed.

## Variants
Binary sun/moon swap (no blend): simpler implementation, but transitions are hard cuts at exact midday/midnight.
