---
title: CapsuleCollider Direction Axis Cheatsheet
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- collider
- capsule
- axis
applies_to:
- unity-projects
source: ''
severity: medium
suggested-category: engine/lessons
time_lost: ''
---

# CapsuleCollider Direction Axis Cheatsheet

## Problem
`CapsuleCollider.direction` is an integer (0/1/2) with no labels in the script API, and the default (1 = Y-axis, vertical) is wrong for most lying objects like fallen tree trunks. Trunk levitates or passes through ground.

## Root cause
`direction` parameter meaning:
- 0 = X-axis (horizontal - good for: fallen trunk, lying cylinder, ship hull)
- 1 = Y-axis (default vertical - good for: character controller, standing tree, pole)
- 2 = Z-axis (forward - good for: torpedo, projectile, forward-pointing object)

For a fallen trunk exported from Blender with its length along the X-axis: direction must be 0.

## Solution
Cheatsheet for TT objects:
- Fallen trunk after tree fall: `direction = 0`, `radius = 0.15`, `height = 3.5`
- Standing tree / character: `direction = 1`, `radius = 0.5`, `height = 5`
- Projectile / forward object: `direction = 2`

To determine the right axis: check which Blender axis is the long axis of the mesh, then apply after FBX axis remap (Blender Y → Unity Z, Blender Z → Unity Y).

Never hardcode collider values from code when the prefab already has a correctly configured CapsuleCollider - see [[script-overrides-prefab-inspector-values]].

## What didn't work
Default direction=1 (vertical) on a lying fallen trunk - capsule points up, trunk levitates.

## Transferability
Any Unity project with non-vertical capsule shapes. The cheatsheet is universal. The Blender-to-Unity axis remap detail is specific to Blender pipelines.

## Related
- [[script-overrides-prefab-inspector-values|Script overrides prefab Inspector values]]
- [Self-collision compound BoxColliders](self-collision-compound-colliders-ignore.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[dynamic-rigidbody-no-nonconvex-meshcollider|Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh]] - wspolne: collider, physics
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] - wspolne: collider, physics
- [[20260710-2115-collider-from-first-meshfilter-antipattern|Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu]] - wspolne: collider, physics
- [[20260622-0950-unity-map-boundary-from-bounds|Invisible map boundary from live Renderer.bounds + foot-only gap via IgnoreCollision]] - wspolne: collider, physics
<!-- /POWIAZANE:auto -->
