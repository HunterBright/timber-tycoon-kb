---
title: bake_space_transform + Linked Duplicates = 90° Rotation Injection
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- fbx
- export
- rotation
- linked-duplicates
applies_to:
- unity-projects
- blender-pipelines
source: ''
severity: high
time_lost: ~1 session
suggested-category: engine/lessons
---

# bake_space_transform + Linked Duplicates = 90° Rotation Injection

## Problem
Using `bake_space_transform=True` in FBX export combined with linked duplicates injects a 90° rotation into instances. In Unity, instances appear visually rotated 90° from the original, even though the Blender scene looks correct.

## Root cause
- Linked duplicates share mesh data but have independent transforms
- `bake_space_transform=True` bakes each object's transform into mesh data on export
- The two mechanisms interact: one duplicate's transform overrides others' visual orientation during bake

## Solution
- Use `PLAIN_AXES` empties as slot markers (e.g., `Fill_N_Slot_M`) instead of mesh duplicates
- Empties define slot positions; instantiate actual prefab at runtime via `PrefabUtility.InstantiatePrefab`
- Coordinate remap when transferring empty positions from Blender to Unity: `(X,Y,Z) → (X,Z,-Y)` + reset rotation to identity before instantiation

## What didn't work
Linked duplicates as slot markers - the rotation injection is baked into the mesh data and has no simple in-Unity fix.

## Transferability
Any Blender FBX pipeline that uses linked duplicates for repeated elements (furniture slots, weapon racks, tile grids) will hit this. Workaround applies to any engine receiving the FBX.

## Related
- [Procedural textures must be baked](procedural-textures-need-bake.md)
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: export, fbx, blender
- [[fbx-export-standard-settings-blender-to-unity|FBX Export Standard Settings (Blender → Unity)]] - wspolne: export, fbx, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] - wspolne: fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: fbx, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
