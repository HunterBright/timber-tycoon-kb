---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, urp, grass, gpu-instancing, foliage, performance, rendering, low-poly]
date: 2026-06-23
status: draft
---

# Performant stylized grass: textured cards + GPU instancing (no GameObjects)

## Problem
Covering a large stylized map with lush grass. Two failed naive approaches:
- **Geometry tufts as prefab instances** — flat-shaded low-poly grass models import into Unity with FAR more verts than Blender reports (flat shading + vertex-color CORNER domain splits every face's corners → ~2× the Blender count). Thousands of GameObjects also blow draw calls, scene size and load time. A real carpet over ~150k m² is infeasible on the vert budget.
- It also just looked faceted/sparse — the art director wanted painterly textured grass (à la Tavern Manager Simulator).

## Pattern (what worked)
Render grass as **textured alpha-clipped CARDS** (a few crossed quads) drawn in bulk via **GPU instancing**, with NO per-blade GameObjects:

1. **Card mesh** (generate in code): 3 quads crossed at 60° around Y, ~0.5 m, UV 0–1, **normals pointing UP (+Y)** so grass lights softly from the sky instead of going dark on the shaded side. ~12 verts.
2. **Texture**: one painterly grass-blade alpha PNG (blades' bases at the bottom edge V=0, transparent background), authored in Blender (model blades → vertex-color → ortho render with `film_transparent`).
3. **Shader** (`Custom/GrassCardInstanced`, URP Forward+): `_BaseMap` + `clip(a-_Cutoff)`, `Cull Off`, SH ambient + main light + ShadowCaster pass, **`#pragma multi_compile_instancing` + `UNITY_VERTEX_INPUT_INSTANCE_ID` + `UNITY_SETUP_INSTANCE_ID`** (REQUIRED for instancing; URP/Lit & most custom shaders DON'T have it by default). Bonus: gentle world-pos+`_Time` wind sway; vertical gradient.
4. **Bake step (editor)**: a scatter tool raycasts placement (slope/road/exclusion filters), groups instances into ~25 m **spatial cells**, and writes a compact **binary blob** (`.bytes` TextAsset): per cell `{bounds, [pos, yRot, scale]...}`. A YAML ScriptableObject of 100k+ Matrix4x4 would bloat git/load — use a binary blob, build matrices at load.
5. **Runtime renderer** (`MonoBehaviour`, `[ExecuteAlways]` so it draws in editor/Scene-View/shot cameras too): subscribe to `RenderPipelineManager.beginCameraRendering`; per camera, for each cell do distance-cull (`maxDrawDistance`) + frustum-cull (`GeometryUtility.TestPlanesAABB`), then `Graphics.RenderMeshInstanced(rp, mesh, 0, cell.matrices, ≤1023, start)` with `rp.camera = cam`. Persists via normal scene serialization (no ISaveable needed; it just references the blob/mesh/material).

## Why it's good
- Verts: card ~12 v; only visible cells (within `maxDrawDistance`) render → tiny per-frame cost even with 100k+ blades total.
- Draw calls: ~tens of `RenderMeshInstanced` calls for ALL grass (vs thousands of objects).
- Scene stays light (one renderer + one blob asset, not 100k GameObjects).

## Gotchas
- **Anti-pop-in**: cell-level culling makes whole 25 m cells appear as slabs. Fix in the shader: fade by scaling vertex Y from 0→1 over a distance band (`_FadeStart`→`_FadeEnd` = `maxDrawDistance`) so blades GROW out of the ground as the camera nears. No hard pop.
- **Editor render-to-texture / Scene View**: pass `camera` per-camera in `beginCameraRendering` so every camera (including temp screenshot cams) gets grass culled correctly.
- `Graphics.RenderMeshInstanced` (Unity 6) over the older `DrawMeshInstanced`; still batch at ≤1023 per call.
- Material MUST have `enableInstancing = true` AND the shader the instancing pragmas — both, or it silently doesn't instance.
- Density gradients (e.g. thin near roads, even-but-short in a work zone) are cheap: multiply a per-candidate ACCEPTANCE PROBABILITY by a spatial factor before adding to the blob; spacing hash still prevents overlap.

## Don't instance everything
Bushes/flowers/trees stayed as ordinary GameObjects (few hundred) — SRP Batcher handles them. Only the tens-of-thousands element (grass) needs this. If accents also balloon, give their shader the same instancing treatment.
