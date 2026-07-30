---
title: Pivot/geometry direction of an FBX must match what a procedural placement tool assumes
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-26'
project: Kerf - Sawmill Tycoon
tags:
- unity
- blender
- fbx
- pivot
- procedural-placement
- editor-tooling
- fence
- off-by-one
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Pivot/geometry direction of an FBX must match what a procedural placement tool assumes

## Severity
Medium — visually broken output (gaps/overhangs), but localized and easy to miss because straight runs still look fine.

## Context
A Unity editor tool (`BuildZoneFences.cs`) tiles a `Fence_Rails` FBX between posts around a
rectangular zone. It placed each rail at the *start* of a span, set `yaw` so local **+X** runs along
the edge, and stretched `localScale.x` to the span length — i.e. it assumed the rail geometry runs
**+X starting from the pivot**.

The actual model had its pivot at the **+X end** with geometry extending in **−X** (measured bounds:
`minX=-2.5, maxX=0.0`, pivot at `x=0`). So every rail extended *backwards*, shifting the whole row by
one full span: a rail stub poked out past the "start" corner of each edge, and the "end" corner was
left with a ~2.5 m gap. Worst at the corner that is the *end* of two edges (open on both sides).

## Lesson
When a tool places/scales an FBX procedurally, the model's **pivot location and geometry direction**
are an implicit contract. A model that "looks centered" in Blender can export with pivot at an end and
geometry on the negative axis. Straight tiled runs hide it (spans still abut); the error only shows at
ends/corners/junctions.

## How to catch it fast
- Read the prefab's combined renderer bounds before trusting placement math:
  `PrefabUtility.LoadPrefabContents(path)` → encapsulate `Renderer.bounds` → check `min/max/center`.
  If `center.x != 0` or `min.x != 0`, the pivot is not the start-of-geometry the tool assumes.
- Verify numerically instead of by eye: for every placed instance compute both endpoints
  (`transform.position` and `transform.TransformPoint(localFarEnd)`) and assert they stay inside the
  intended bounds and that each corner/junction has an endpoint within ε. This is more reliable than a
  screenshot (and the Unity MCP `Capture2DScene` tool renders nothing for 3D meshes anyway; the 3D
  `MultiAngleSceneView` needs legacy int instance IDs which are compile-errors in Unity 6000.5+).

## Fix shape
Cheapest fix is in the *tool*, not the model: place the instance at the span **end** node (and end
ground height) so the −X geometry fills the span. Rotation/scale math was already correct — only the
anchor node was wrong. (Re-exporting the model with +X-from-origin geometry also works but needs a
Blender round-trip + re-bake + prefab rebuild, so prefer the one-line tool fix when the model is used
by a single tool.)

## Gotcha
The colliders in the same tool were placed at span *midpoints*, so collision was correct even while the
visible rails were offset — the fence blocked properly but looked wrong. Don't assume "collision works
⇒ geometry is right."
