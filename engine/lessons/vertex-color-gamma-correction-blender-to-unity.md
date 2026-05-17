---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [blender, unity, vertex-colors, gamma, srgb, linear]
severity: high
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
---

# Vertex Color Gamma Correction Blender → Unity

## Problem
Vertex colors assigned in Blender appear noticeably darker in Unity. Bright grass green in Blender becomes muddy olive in Unity. Every terrain regeneration requires multiple iterations of "darker, no darker, no lighter" until you discover the gamma issue.

## Root cause
Blender stores vertex colors in linear space (0–1 floats, no gamma). Unity shaders (URP Lit, Custom/VertexColorLit) expect sRGB. The gamma mismatch causes all vertex colors to read darker than designed.

## Solution
Apply gamma correction in the Blender Python export script before writing vertex colors:

```python
def linear_to_srgb(c):
    if c <= 0.0031308: return c * 12.92
    return 1.055 * (c ** (1/2.4)) - 0.055
```

Apply per-channel before writing to the vertex color attribute.

Simpler approximation (close enough for low-poly): `c ** (1/2.2)`

Verify in Unity Play Mode Game view — Editor Scene view can lie.

## What didn't work
Assigning vertex colors directly from linear Blender values — all colors darker than designed.

## Transferability
Any pipeline that carries vertex colors from Blender to Unity (or any sRGB-space engine) will hit this. Bake the conversion into the export script once; never manually compensate colors again.

## Related
- [Desaturated colors for low-poly](desaturated-colors-for-low-poly.md)
- [Procedural textures must be baked](procedural-textures-need-bake.md)
