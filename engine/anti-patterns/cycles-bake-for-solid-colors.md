---
name: cycles-bake-for-solid-colors-antipattern
description: Smart UV Project + Cycles bake for solid color regions fails — black output, blurred edges, ghost UV islands. Use numpy atlas + manual UV instead.
metadata:
  type: anti-pattern
  project: timber-tycoon
  suggested-category: engine/anti-patterns
  tags: [blender, baking, uv, anti-pattern, static-props]
  severity: high
  time_lost: 3h (PelletBag debugging)
  date: 2026-05-17
  status: draft
  applies_to: [blender-pipelines]
---

# ANTI-PATTERN: Cycles Bake for Solid Color Regions

## The Trap
You have a static prop (bag, box, container) with several flat-colored regions. You run Smart UV Project on the whole mesh, set up a Cycles bake, and expect a clean atlas of solid colors. Instead:
- Solid color regions bake **BLACK** — Cycles treats them as unlit surfaces and ambient occlusion overpowers
- Edges between UV islands are **blurred** — Cycles sampling averages neighboring pixels at island borders
- **Ghost UV islands** — Smart UV Project creates micro-islands invisible in viewport; baked output has bleed artifacts from them

## Why It Fails
Cycles bake is designed for shaded surfaces: diffuse reflection, ambient occlusion, normal maps. Flat solid-color regions have no lighting information — Cycles doesn't know what color to output for "no light at all." The result is ambient-occlusion-dominated black.

## Correct Approach
For static props with solid color regions, use **numpy-based manual atlas** in Blender Python:

```python
import numpy as np
from PIL import Image

# Create atlas PNG with explicit color blocks
atlas = np.zeros((1024, 1024, 4), dtype=np.uint8)
# Label region (left 75%): paste label PNG
atlas[:, :768] = label_array
# Body region (right-bottom): solid off-white
atlas[512:, 768:] = [230, 225, 215, 255]
# Accent region (right-top): solid tan
atlas[:512, 768:] = [180, 158, 120, 255]

Image.fromarray(atlas).save("T_Prop_Atlas.png")
```

Then assign UVs per-face to the correct atlas region manually:
- Label faces → U range 0..0.75
- Body faces → U range 0.875, V range 0..0.25
- Accent faces → U range 0.875, V range 0.75..1.0

**Reserve Cycles bake for:** materials with actual shader complexity — procedural wood grain, normal maps, ambient occlusion on complex geo, worn metal textures. For those, Cycles bake is correct and powerful.

## Concrete case
PelletBag v1–v7 used Smart UV Project + Cycles bake. Result: white label area baked as black, accent strip invisible, 3 hours of iteration with no improvement. v8 switched to numpy atlas — result: correct in one attempt.

## See also
[[single-material-atlas-for-static-props]], [[procedural-textures-need-bake]]
