---
name: mesh-collider-on-roads-stackable
description: MeshCollider on generated roads enables raycasting onto road surfaces, allowing new road waypoints to stack on existing roads
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, road, collider, stacking, raycast]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
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
- `MeshCollider.convex = false` on road meshes: non-convex colliders can't participate in dynamic Rigidbody collision — acceptable, roads are static
- Performance: MeshCollider cooking (baking collision mesh) takes time on mesh changes. For procedural tools, cook once after generation is complete
- Convex mesh limit: each MeshCollider must be convex if set to convex=true. Roads with many waypoints are never convex — keep `convex = false`

## Variants
- **Terrain collider only**: skip road MeshColliders, just elevate waypoints manually. Simpler but no stacking support
- **NavMesh baking**: roads naturally become part of NavMesh walkable area when they have MeshColliders + correct layer — NPC pathfinding bonus

See also: [[catmull-rom-spline-road-mesh]], [[flatten-terrain-under-road-smoothstep]]
