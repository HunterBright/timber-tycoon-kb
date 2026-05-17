---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, terrain, heightmap, cliffs, caves, level-design]
severity: high
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Separate-Objects Mapping Rule (Heightmap Limitations)

## Problem
Trying to represent cliffs, overhangs, caves, or tunnels in a heightmap produces degenerate triangles and broken geometry. Heightmaps cannot represent two Y values at the same XZ coordinate.

## Root cause
A heightmap is a 2D function Y = f(X, Z). Mathematically it can only have one height value per horizontal position — it cannot represent overhangs (cliff sticking out), caves (two Y values at same XZ), or any underpass structure.

## Solution
Build non-heightmap geometry as separate FBX objects placed on the heightmap:
- Cliff_Waterfall.fbx, Waterfall.fbx, Bridge.fbx, Mountains.fbx, River_Cave.fbx — all separate FBX objects
- Each owns its mesh, materials, colliders
- Setup scripts orchestrate placement: `RebuildFullMap.cs` calls `SetupCliff.cs`, `SetupBridge.cs`, etc.
- Lock positions once placed: `// DO NOT MOVE` convention (see [[do-not-move-hardcoded-positions]])
- Heightmap = base terrain only (no vertical complexity)

## What didn't work
Attempting to model cliffs and caves in heightmap — sub-zero Y values, degenerate triangles, broken colliders.

## Transferability
Any open-world game with interesting terrain features (caves, cliffs, bridges, tunnels) must use the same split approach: heightmap for gentle terrain, separate mesh objects for vertical features. The pattern applies to any engine and any terrain tool.

## Related
- [Cliff + Waterfall hidden cave pattern](../patterns/cliff-waterfall-hidden-cave.md)
- [Mountains hierarchy](../patterns/mountains-hierarchy-front-and-backdrop.md)
