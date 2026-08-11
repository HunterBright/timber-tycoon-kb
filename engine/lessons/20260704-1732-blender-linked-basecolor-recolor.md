---
title: Recoloring a Blender material whose Base Color is LINKED does nothing via default_value
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- blender
- materials
- principled-bsdf
- procedural
- recolor
- python
- node-links
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Recoloring a Blender material whose Base Color is LINKED does nothing via default_value

## Symptom
Setting `bsdf.inputs['Base Color'].default_value = (r,g,b,1)` in a Python script appears to
succeed but the object's visible color never changes.

## Root cause
If the Principled BSDF's Base Color socket `is_linked` (fed by a MIX / TEX_IMAGE / ColorRamp /
Noise node - typical of any procedural "grime/rust" material or a baked-atlas material), the
`default_value` is ignored entirely. The rendered color comes from the upstream node graph, not
from the socket's default. No error is raised - it silently no-ops.

## Fix / workflow
For a flat recolor (e.g. tier variants, review scenes): don't try to edit the linked socket.
Instead build a NEW flat Principled material (`bpy.data.materials.new`, single Base Color +
Metallic + Roughness, no procedural nodes) and reassign it to the object's material slot
(`slot.material = new_mat`). This also happens to read as the "cleanest" end of a
worn→pristine ladder, so it doubles as the high-tier finish.

## Diagnosis tip
Before recoloring, enumerate each material: read `bsdf.inputs['Base Color'].is_linked` and the
node types (`set(n.type for n in mat.node_tree.nodes)`). If you see MIX/VALTORGB/TEX_NOISE/
TEX_IMAGE, the base color is procedural - plan to replace, not tweak.

## Related gotcha (same session)
A machine that has been baked to a single texture atlas has ALL its parts sharing ONE material
(base color linked to the atlas image). Per-part recolor is impossible without first
re-splitting: reassign fresh flat materials to individual objects by object identity. The
original per-part materials may still exist in the file but orphaned (kept alive only by
fake-user, assigned to nothing) - don't assume the named materials are the ones actually in use;
enumerate object→slot→material first.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260730-1710-blender-materials-clear-resets-face-indices|Mesh.materials.clear() zeruje material_index na ściankach]] - wspolne: python, materials, blender
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: python, blender
- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] - wspolne: materials, blender
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] - wspolne: materials, blender
- [[20260726-1535-blender-addon-parametryczny-suwaki|Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda]] - wspolne: python, blender
- [[procedural-textures-cycles-commercial|Procedural Textures in Blender Cycles (Commercial Release Rationale)]] - wspolne: procedural, blender
<!-- /POWIAZANE:auto -->
