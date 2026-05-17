---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, physics, colliders, compound, rigidbody, vehicle]
severity: high
time_lost: "~40 min"
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Self-Collision Compound BoxColliders → Physics.IgnoreCollision

## Problem
Vehicles with compound BoxColliders (Cabin + Flatbed + body parts) explode or launch into the air on spawn. The Rigidbody violently reacts to internal forces that weren't there in the editor.

## Root cause
Compound colliders on the same Rigidbody are NOT automatically excluded from colliding with each other. Unity's physics engine detects overlap between Cabin BoxCollider and body BoxCollider, applies separation forces — resulting in the vehicle launching or jittering violently at spawn.

## Solution
At Awake, iterate all child `Collider` components and call `Physics.IgnoreCollision` for every pair:

```csharp
void Awake() {
    var cols = GetComponentsInChildren<Collider>();
    for (int i = 0; i < cols.Length; i++)
        for (int j = i + 1; j < cols.Length; j++)
            Physics.IgnoreCollision(cols[i], cols[j], true);
}
```

Caveat: only applies to colliders on the SAME Rigidbody. Child Rigidbodies need separate handling.

## What didn't work
Assuming Unity auto-excludes intra-object collisions for compound setups.

## Transferability
Any Unity project using compound collider hierarchies on a single Rigidbody — robots, vehicles, modular characters, destructible objects — needs this fix. Standard boilerplate for any Awake that sets up physics-driven objects with multiple collider children.

## Related
- [Forward axis quirk](forward-axis-blender-fbx-quirk.md)
- [Vehicle interaction zones as triggers](../patterns/vehicle-interaction-zones-as-triggers.md)
