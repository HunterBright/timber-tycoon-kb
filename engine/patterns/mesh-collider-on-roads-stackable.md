---
title: MeshCollider on Roads = Stackable
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- road
- collider
- stacking
- raycast
applies_to: []
source: ''
suggested-category: engine/patterns
---

# MeshCollider on Roads = Stackable

## When to use
Procedural road generation system where new roads should be able to cross or sit on top of existing roads (sidewalk on asphalt, path over bridge deck, intersection).

## Steps

**After generating road mesh, assign to MeshCollider:**
```csharp
var col = roadGO.GetOrAddComponent<MeshCollider>();
col.sharedMesh = generatedMesh;
col.convex = false; // road meshes are non-convex
```

**Road layer assignment:**
- Roads on layer `Road` (layer 6 in TT)
- Terrain on layer `Default` (layer 0)

**Waypoint placement raycast includes Road layer:**
```csharp
// RoadTool Scene View click handler:
LayerMask roadPlacementMask = LayerMask.GetMask("Default", "Road", "Terrain");
if (Physics.Raycast(editorRay, out hit, Mathf.Infinity, roadPlacementMask))
    waypoints.Add(hit.point);
```

**Result:**
- Click on terrain → waypoint at terrain height
- Click on existing road → waypoint at road height (sits on road surface)
- Click on bridge → waypoint on bridge deck
- New road follows the surface correctly without manual Y adjustment

**Crossing pattern:**
- Sidewalk on asphalt road: both have MeshColliders. Sidewalk waypoints clicked on asphalt surface → sidewalk sits on top of asphalt.
- Dirt path over concrete road: path waypoints hit concrete, path mesh generated at concrete height.

## Why this works
MeshCollider makes roads part of the raycast-able physics scene. Waypoint tool raycasts into this scene to find hit positions. Stack N roads = each layer is discoverable by the next.

## Trade-offs
- `MeshCollider.convex = false` on road meshes: non-convex colliders can't participate in dynamic Rigidbody collision - acceptable, roads are static
- Performance: MeshCollider cooking (baking collision mesh) takes time on mesh changes. For procedural tools, cook once after generation is complete
- Convex mesh limit: each MeshCollider must be convex if set to convex=true. Roads with many waypoints are never convex - keep `convex = false`

## Variants
- **Terrain collider only**: skip road MeshColliders, just elevate waypoints manually. Simpler but no stacking support
- **NavMesh baking**: roads naturally become part of NavMesh walkable area when they have MeshColliders + correct layer - NPC pathfinding bonus

See also: [[catmull-rom-spline-road-mesh]], [[flatten-terrain-under-road]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260609-0830-conform-terrain-to-path-via-per-x-centerline-profile|Conform a mesh terrain to a path by sampling its centreline per-axis (not a fixed scan line, not a single plane)]] - wspolne: road, raycast
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] - wspolne: collider, raycast
- [[20260628-1140-conform-road-mesh-to-edited-terrain|Conforming an existing road/decal mesh to terrain that was edited later]] - wspolne: road, raycast
- [[20260724-1817-diegetic-button-overlap-steal|Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik]] - wspolne: collider, raycast
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] - wspolne: collider, raycast
<!-- /POWIAZANE:auto -->
