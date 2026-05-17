---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [blender, fbx, export, rotation, linked-duplicates]
severity: high
time_lost: "~1 session"
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
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
Linked duplicates as slot markers — the rotation injection is baked into the mesh data and has no simple in-Unity fix.

## Transferability
Any Blender FBX pipeline that uses linked duplicates for repeated elements (furniture slots, weapon racks, tile grids) will hit this. Workaround applies to any engine receiving the FBX.

## Related
- [Procedural textures must be baked](procedural-textures-need-bake.md)
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)
