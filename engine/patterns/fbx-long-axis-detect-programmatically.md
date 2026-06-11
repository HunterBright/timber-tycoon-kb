---
type: pattern
project: Timber_Tycoon
suggested-category: engine/patterns
tags: [unity, blender, fbx, axis, viewmodel, bounds, orientation]
date: 2026-06-11
status: validated
---

# Don't assume an FBX mesh's axis — detect the longest axis programmatically from bounds

## Problem
Another case in the "Blender FBX axis quirk" family (previously: car forward = -transform.right, bake_space_transform on empties). This time: the stacked-carry viewmodel assumed a log "lies" along local Z. In reality all 4 log prefabs (Spruce/Birch/Oak/Maple) have their length along **local Y** → carried logs rendered vertically.

## Pattern
When code has to orient an arbitrary mesh "along" something (carry viewmodel, placing on a rack, cargo on a flatbed), do NOT hardcode the axis per prefab. Instead, at spawn/attach time:

1. Gather the bounds of all `MeshFilter.sharedMesh.bounds` in the root's local space:
   `Matrix4x4 toRoot = root.worldToLocalMatrix * mf.transform.localToWorldMatrix;` — transform the 8 corners of each bounds, encapsulate.
2. Multiply the resulting size by `root.localScale` (scale is applied before rotation, so it affects the visually longest axis).
3. Longest component → the axis; `Quaternion.FromToRotation(axis, desiredDirection)` as localRotation (for rotationally symmetric shapes, FromToRotation's roll ambiguity does no harm).

## Caveat (important)
Apply the axis mapping ONLY to clearly elongated objects. For roughly cubic objects (stumps: axis differences on the order of millimeters), the "longest axis" is measurement noise — the mapping randomly tips the object over. Filter by object type or by ratio (e.g. longest/second < 1.3 → don't map).

## Why this is future-proof
Zero code changes for new assets: every future species/prefab orients itself, regardless of how Blender exported its axes.

## See also
[[forward-axis-blender-fbx-quirk]], [[bake-space-transform-linked-duplicates-rotation-bug]], [[stacked-carry-system-camera-viewmodel]], [[zero-code-changes-philosophy]]
