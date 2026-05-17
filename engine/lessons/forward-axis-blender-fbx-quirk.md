---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, blender, fbx, transform, vehicle, forward-axis]
severity: high
time_lost: "multiple debugging sessions"
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
---

# Forward Axis = -transform.right (Blender FBX Quirk)

## Problem
Vehicles modeled in Blender and exported via FBX have their local forward axis = `-transform.right` in Unity, not `transform.forward`. The car drives sideways or refuses to move. Steering and parking alignment are broken.

## Root cause
Blender's default Y-forward axis maps to Unity's -X axis after the FBX coordinate conversion (`axis_forward='-Z'`, `axis_up='Y'`). The mesh front ends up pointing along Unity's -X (negative right) direction.

## Solution
Define a forward axis property and use it everywhere instead of `transform.forward`:
```csharp
public Vector3 ForwardAxis => -transform.right;
```

Affects:
- `VehicleController` (AddForce direction)
- `NPCVehicle` (movement direction)
- `NPCParkingPDController` (slot forward alignment)

Detection: if car drives sideways on first test → check this first. It's always this.

Alternative (rejected): rotate model 90° in Blender before export — breaks existing prefabs.

## What didn't work
Using `transform.forward` directly — car drives sideways.

## Transferability
Any Blender-authored vehicle, character, or directional prop exported via FBX will have this quirk unless the Blender artist explicitly rotates the root object 90° in Y before export. Document this as a project convention; every new vehicle developer needs to know.

## Related
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)
- [Self-collision compound BoxColliders](self-collision-compound-colliders-ignore.md)
