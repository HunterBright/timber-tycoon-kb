---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [blender, fbx, materials, baking, unity, urp]
severity: high
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
---

# Procedural Textures Must Be Baked Before FBX Export

## Problem
Blender procedural shaders (Wave/Voronoi/Noise + ColorRamp → Principled BSDF) don't carry through FBX export. Unity receives pink or gray materials instead of the designed look. No error is shown — materials simply arrive wrong.

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
