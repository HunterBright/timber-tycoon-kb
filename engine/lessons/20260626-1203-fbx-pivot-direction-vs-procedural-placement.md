---
title: Pivot/geometry direction of an FBX must match what a procedural placement tool assumes
type: lesson
status: active
confidence: medium
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
promoted: '2026-07-30'
---

# Pivot/geometry direction of an FBX must match what a procedural placement tool assumes

## Severity
Medium - visually broken output (gaps/overhangs), but localized and easy to miss because straight runs still look fine.

## Context
A Unity editor tool (`BuildZoneFences.cs`) tiles a `Fence_Rails` FBX between posts around a
rectangular zone. It placed each rail at the *start* of a span, set `yaw` so local **+X** runs along
the edge, and stretched `localScale.x` to the span length - i.e. it assumed the rail geometry runs
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
ground height) so the −X geometry fills the span. Rotation/scale math was already correct - only the
anchor node was wrong. (Re-exporting the model with +X-from-origin geometry also works but needs a
Blender round-trip + re-bake + prefab rebuild, so prefer the one-line tool fix when the model is used
by a single tool.)

## Gotcha
The colliders in the same tool were placed at span *midpoints*, so collision was correct even while the
visible rails were offset - the fence blocked properly but looked wrong. Don't assume "collision works
⇒ geometry is right."

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[asset-origin-bottom-center-convention|Asset Origin at Bottom-Center Convention]] - wspolne: pivot, blender
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] - wspolne: fbx, blender
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] - wspolne: fbx, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: fbx, blender
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] - wspolne: fbx, blender
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] - wspolne: fbx, blender
<!-- /POWIAZANE:auto -->
