---
title: Don't assume an FBX mesh's axis - detect the longest axis programmatically from bounds
type: pattern
status: verified
confidence: high
verified: ''
date: '2026-06-11'
project: Timber_Tycoon
tags:
- unity
- blender
- fbx
- axis
- viewmodel
- bounds
- orientation
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Don't assume an FBX mesh's axis - detect the longest axis programmatically from bounds

## Problem
Another case in the "Blender FBX axis quirk" family (previously: car forward = -transform.right, bake_space_transform on empties). This time: the stacked-carry viewmodel assumed a log "lies" along local Z. In reality all 4 log prefabs (Spruce/Birch/Oak/Maple) have their length along **local Y** → carried logs rendered vertically.

## Pattern
When code has to orient an arbitrary mesh "along" something (carry viewmodel, placing on a rack, cargo on a flatbed), do NOT hardcode the axis per prefab. Instead, at spawn/attach time:

1. Gather the bounds of all `MeshFilter.sharedMesh.bounds` in the root's local space:
   `Matrix4x4 toRoot = root.worldToLocalMatrix * mf.transform.localToWorldMatrix;` - transform the 8 corners of each bounds, encapsulate.
2. Multiply the resulting size by `root.localScale` (scale is applied before rotation, so it affects the visually longest axis).
3. Longest component → the axis; `Quaternion.FromToRotation(axis, desiredDirection)` as localRotation (for rotationally symmetric shapes, FromToRotation's roll ambiguity does no harm).

## Caveat (important)
Apply the axis mapping ONLY to clearly elongated objects. For roughly cubic objects (stumps: axis differences on the order of millimeters), the "longest axis" is measurement noise - the mapping randomly tips the object over. Filter by object type or by ratio (e.g. longest/second < 1.3 → don't map).

## Why this is future-proof
Zero code changes for new assets: every future species/prefab orients itself, regardless of how Blender exported its axes.

## See also
[[forward-axis-blender-fbx-quirk]], [[bake-space-transform-linked-duplicates-rotation-bug]], [[stacked-carry-system-camera-viewmodel]], [[zero-code-changes-philosophy]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[flatten-must-be-baked-into-geometry-when-code-forces-uniform-scale|Flatten Must Be Baked Into Geometry When Code Forces Uniform Scale]] - wspolne: axis, fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: orientation, fbx, blender
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: orientation, fbx, blender
- [[20260531-0934-humanoid-orientation-from-armature-not-bbox|Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread - a T-pose arm span can beat true height]] - wspolne: orientation, blender
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: fbx, blender
- [[blender-mcp-interactive-remodel-loop|Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
