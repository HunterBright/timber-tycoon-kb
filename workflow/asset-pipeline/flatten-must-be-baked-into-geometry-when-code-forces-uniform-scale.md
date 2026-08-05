---
title: Flatten Must Be Baked Into Geometry When Code Forces Uniform Scale
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-05-29'
project: Kerf - Sawmill Tycoon
tags:
- blender
- unity
- scale
- geometry
- fbx
- axis
- spawn-rotation
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Flatten Must Be Baked Into Geometry When Code Forces Uniform Scale

## Problem
A prop needs a non-uniform proportion (a flattened lump of soil - wider than tall). Setting that via the transform's non-uniform scale gets silently wiped, because the spawn code forces a UNIFORM scale:

```csharp
clump.transform.localScale = Vector3.one * dirtClumpScale;   // 1.5 - uniform, overwrites any non-uniform scale
```

Any `(x, y*0.5, z)` flatten applied as transform scale is gone the instant the object spawns.

## Root cause
`Vector3.one * k` is uniform on all three axes. It overwrites whatever scale the prefab/mesh had, so proportion baked into transform scale cannot survive. Proportion has to live where uniform scaling can't erase it: in the vertex positions.

## Solution
**Bake the flatten into the mesh vertices at object scale (1,1,1).** In Blender, multiply the verts' Z by the flatten factor (here ×0.52), apply all transforms so object scale is exactly (1,1,1), then export. Uniform ×1.5 at spawn then scales the already-flat mesh uniformly and the proportion is preserved.

Measured result - DirtClump:
- Blender dims X/Y/Z = 0.2294 / 0.2335 / **0.1365** (flattened on Blender Z), object scale (1,1,1).

### Axis note (Blender Z → Unity Y)
Flatten on **Blender Z**. With the standard FBX export (`axis_up='Y'`, `axis_forward='-Z'`, see [[fbx-export-standard-settings-blender-to-unity]]), Blender Z maps to **Unity Y**, so the flat axis becomes vertical and the prop **lies flat on the ground**. Verified in Unity: bounds X/Y/Z = 0.2294 / **0.1365** / 0.2335 → smallest extent is Y → imported lying flat, not on edge.

### Pair with yaw-only spawn rotation
A flattened object spawned with full `Random.rotation` will frequently land **tipped on its edge** - random 3D orientation ignores the fact that it has a "lying flat" pose. Use yaw-only rotation so it always lies flat and only spins in the horizontal plane:

```csharp
// instead of Random.rotation
Quaternion.Euler(0f, Random.Range(0f, 360f), 0f)
```

Apply this in every spawn branch (prefab branch AND any fallback primitive branch) for consistent behaviour.

## What didn't work
Relying on a non-uniform transform scale for the flatten - wiped by `Vector3.one * scale` at spawn. Also: keeping `Random.rotation` after flattening - clumps spawned standing on edge instead of lying flat.

## Transferability
Any engine/spawn system that forces uniform scale on instantiated objects (very common for "place N copies" code). If the art needs a non-uniform silhouette, bake it into geometry, mind the Blender-Z→Unity-Y axis swap, and constrain spawn rotation to the axis that keeps the intended resting pose.

## Related
- [[fbx-export-standard-settings-blender-to-unity]]
- [[script-overrides-prefab-inspector-values]]
- [[asset-origin-bottom-center-convention]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: scale, fbx, blender
- [[fbx-long-axis-detect-programmatically|Don't assume an FBX mesh's axis - detect the longest axis programmatically from bounds]] - wspolne: axis, fbx, blender
- [[20260531-1705-normalize-assetpack-scale-via-modelimporter|Normalize Inconsistent Asset-Pack Scale at the Source (ModelImporter.globalScale)]] - wspolne: scale, fbx
- [[20260802-0950-mierz-wyeksportowany-plik-nie-policzona-skale|20260802-0950-mierz-wyeksportowany-plik-nie-policzona-skale]] - wspolne: fbx, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
