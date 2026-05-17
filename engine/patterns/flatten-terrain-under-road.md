---
name: flatten-terrain-under-road-smoothstep
description: Terrain flattening under roads — smoothstep blend zone gracefully transitions from flat road surface to natural terrain contour
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, road, terrain, mesh-modification, smoothstep]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Flatten Terrain Under Road (Smoothstep Blend)

## When to use
Procedural roads on a terrain mesh. Without terrain flattening, roads hover over bumps or sink into hills. Without a blend zone, there's a hard seam where road meets ground.

## Steps

**Algorithm — iterate every terrain vertex near the road:**
```csharp
void FlattenTerrainUnderRoad(Vector3[] terrainVertices, List<Vector3> splinePoints,
    float roadWidth, float blendZone, float roadY) {

    for (int i = 0; i < terrainVertices.Length; i++) {
        float dist = MinDistanceToSpline(terrainVertices[i], splinePoints);
        if (dist > roadWidth + blendZone) continue; // outside blend area

        // t = 0 at road center, 1 at blend zone edge
        float t = Mathf.Clamp01((dist - roadWidth) / blendZone);
        float smoothT = Mathf.SmoothStep(0f, 1f, t);
        terrainVertices[i].y = Mathf.Lerp(roadY, terrainVertices[i].y, smoothT);
    }
}
```

**Parameters:**
- `roadWidth`: half-width of the road (e.g., 1.5m for a 3m-wide road)
- `blendZone`: transition width on each side (typically 1–2× roadWidth, e.g., 3m for a 3m road)
- `roadY`: target Y elevation of the flat road surface

**Result:** road surface is flat at `roadY`, surrounding terrain transitions smoothly back to natural height over `blendZone` distance.

**After modifying:** refresh `MeshCollider` with modified mesh, save mesh as `.asset` to persist (otherwise modification lost at re-run).

## Why this works
`Mathf.SmoothStep` produces a smooth S-curve (zero derivative at both ends), eliminating any visible edge or pinch where road meets terrain. No hard steps, no visible seam.

## Trade-offs
- Modifies terrain mesh in-place: destructive if not saved. Always save as a named mesh asset before closing the Editor
- `MinDistanceToSpline` is O(N × M) where N = terrain verts, M = spline points — for large terrains, this is slow. Cache spline points, use spatial partition (grid) for lookup
- Blend zones from adjacent roads may conflict where roads cross — apply roads sequentially, not simultaneously

## Variants
- **Unity Terrain flatten:** uses `TerrainData.SetHeights()` — simpler API, works only with Unity Terrain (not custom mesh)
- **Static in Blender:** manually conform road topology to terrain in Blender (face-copy, terrain +2cm offset) — TT uses this for static roads in Roads.fbx, more control

See also: [[catmull-rom-spline-road-mesh]], [[mesh-collider-on-roads-stackable]]
