---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, road, mesh-generation, spline, catmull-rom]
date: 2026-05-17
status: draft
---

# Catmull-Rom Spline + Quad Strip Mesh for Roads

## When to use
Procedural road generation in Unity where designers place waypoints in the Scene View and roads are generated from them. Faster than modeling each road in Blender, especially for iteration.

## Steps

**Catmull-Rom spline through waypoints:**
```csharp
Vector3 CatmullRom(Vector3 p0, Vector3 p1, Vector3 p2, Vector3 p3, float t) {
    return 0.5f * ((2f * p1)
        + (-p0 + p2) * t
        + (2f * p0 - 5f * p1 + 4f * p2 - p3) * (t * t)
        + (-p0 + 3f * p1 - 3f * p2 + p3) * (t * t * t));
}
// Sample: splineResolution = 10 segments between each waypoint pair
```

**Quad strip mesh generation:**
```csharp
// For each spline point i:
//   direction = (point[i+1] - point[i]).normalized
//   right = Vector3.Cross(direction, Vector3.up) * roadWidth * 0.5f
//   left vertex:  point[i] - right
//   right vertex: point[i] + right
// Quad: [left[i], right[i], right[i+1], left[i+1]]
```

**Vertex colors per road type:**

| Type | Color |
|------|-------|
| Dirt path | Brown (0.55, 0.42, 0.28) |
| Asphalt | Dark gray (0.25, 0.25, 0.25) |
| Gravel | Light gray (0.65, 0.60, 0.55) |
| Forest path | Dark earth (0.38, 0.32, 0.22) |
| Sidewalk | Light concrete (0.78, 0.75, 0.70) |

**Perlin noise variation:** per-vertex `color + Mathf.PerlinNoise(x * freq, z * freq) * amplitude` — organic color variation, subtle (amplitude ~0.05).

**Shader:** `Custom/LowPolyVertexColor` — same as terrain.

**5 road types in TT**, each with preset color + `defaultWidth` field in `RoadTypeSO`.

## Why this works
Catmull-Rom produces smooth curves through all control points, not just toward them (unlike Bezier). Quad strip mesh is trivially generated from the spline. Designers iterate in seconds: move a waypoint, press Generate, road updates.

## Trade-offs
- Spline resolution vs. polygon count: `splineResolution = 10` between waypoints means a 100-waypoint road = 1000 quads = 4000 verts — acceptable
- Sharp turns: Catmull-Rom doesn't guarantee perpendicular quads at high curvature. Clamp maximum turn rate between waypoints
- Terrain interaction: road mesh must be elevated above terrain to avoid z-fighting (see [[flatten-terrain-under-road-smoothstep]])

## Variants
- **Bezier spline:** more control over tangents, but waypoints don't sit on the curve
- **Static FBX roads:** model in Blender, export, import. More work per iteration, better for final polished roads

See also: [[flatten-terrain-under-road-smoothstep]], [[mesh-collider-on-roads-stackable]]
