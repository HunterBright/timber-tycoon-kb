---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, physics, collider, capsule, axis]
severity: medium
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# CapsuleCollider Direction Axis Cheatsheet

## Problem
`CapsuleCollider.direction` is an integer (0/1/2) with no labels in the script API, and the default (1 = Y-axis, vertical) is wrong for most lying objects like fallen tree trunks. Trunk levitates or passes through ground.

## Root cause
`direction` parameter meaning:
- 0 = X-axis (horizontal — good for: fallen trunk, lying cylinder, ship hull)
- 1 = Y-axis (default vertical — good for: character controller, standing tree, pole)
- 2 = Z-axis (forward — good for: torpedo, projectile, forward-pointing object)

For a fallen trunk exported from Blender with its length along the X-axis: direction must be 0.

## Solution
Cheatsheet for TT objects:
- Fallen trunk after tree fall: `direction = 0`, `radius = 0.15`, `height = 3.5`
- Standing tree / character: `direction = 1`, `radius = 0.5`, `height = 5`
- Projectile / forward object: `direction = 2`

To determine the right axis: check which Blender axis is the long axis of the mesh, then apply after FBX axis remap (Blender Y → Unity Z, Blender Z → Unity Y).

Never hardcode collider values from code when the prefab already has a correctly configured CapsuleCollider — see [[script-overrides-prefab-inspector]].

## What didn't work
Default direction=1 (vertical) on a lying fallen trunk — capsule points up, trunk levitates.

## Transferability
Any Unity project with non-vertical capsule shapes. The cheatsheet is universal. The Blender-to-Unity axis remap detail is specific to Blender pipelines.

## Related
- [Script overrides prefab Inspector values](../anti-patterns/script-overrides-prefab-inspector.md)
- [Self-collision compound BoxColliders](self-collision-compound-colliders-ignore.md)
