---
title: Procedural Textures Must Be Baked Before FBX Export
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- fbx
- materials
- baking
- unity
- urp
applies_to:
- unity-projects
- blender-pipelines
source: ''
severity: high
suggested-category: engine/lessons
time_lost: ''
---

# Procedural Textures Must Be Baked Before FBX Export

## Problem
Blender procedural shaders (Wave/Voronoi/Noise + ColorRamp → Principled BSDF) don't carry through FBX export. Unity receives pink or gray materials instead of the designed look. No error is shown - materials simply arrive wrong.

## Root cause
FBX format carries texture FILE references, not shader graphs. Procedural shaders exist only in Blender's render context (Cycles/Eevee). Once exported to FBX, the shader data has nowhere to go.

## Solution
- Bake all procedural materials to PNG before export: Albedo + Normal + AO maps
- GPU baking on RTX 4090: ~2-5 min per asset (Cycles)
- Naming: `Mat_{ObjectName}_Bake.png` in `Assets/Models/{Category}/Textures/`
- Unity: URP Lit shader, assign Base Map / Normal / Occlusion from baked PNGs
- File-size guidance: 512×512 for small props, 1024×1024 for buildings/vehicles

## What didn't work
Nieudokumentowane.

## Transferability
Any pipeline that takes models from Blender to a game engine via FBX or GLTF will lose procedural shaders. The bake step is mandatory regardless of engine. This applies to every project using Blender as the modeling tool.

## Related
- [bake_space_transform + linked duplicates rotation bug](bake-space-transform-linked-duplicates-rotation-bug.md)
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260615-2045-blender-voronoi-round-knots|Proceduralne okrągłe sęki w Blenderze: Voronoi F1, nie DISTANCE_TO_EDGE (+ kompensacja proporcji)]] - wspolne: baking, blender
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] - wspolne: materials, urp, blender
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: materials, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: materials, urp
- [[20260713-2145-urp-transparent-material-silent-failure|URP: źle skonfigurowany materiał przezroczysty to CICHA porażka, której wykrywacz magenty nie widzi]] - wspolne: materials, urp
- [[20260719-1605-paper-shell-culling-seethrough|Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię]] - wspolne: materials, urp
<!-- /POWIAZANE:auto -->
