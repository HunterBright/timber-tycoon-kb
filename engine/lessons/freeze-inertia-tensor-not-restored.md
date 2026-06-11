---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, rigidbody, physics, inertia-tensor, vehicle, npc]
severity: high
date: 2026-05-22
status: draft
---

# FreezeAll + automaticInertiaTensor=false Zeroes the Inertia Tensor

## Problem
After an NPC vehicle finishes kinematic reverse-parking, it freezes (`RigidbodyConstraints.FreezeAll`) with `automaticInertiaTensor=false` while the passenger exits. On unfreeze for departure, the car refuses to rotate — it drives as if its rotational inertia is gone, sliding forward without turning.

## Root cause
Setting `automaticInertiaTensor=false` combined with `FreezeAll` zeroes the inertia tensor. Unity does **not** automatically restore the inertia tensor when constraints are released. The Rigidbody is left with a near-zero rotational inertia after unfreeze.

## Solution
Cache the inertia tensor (and tensor rotation) **before** freezing, then restore it explicitly on unfreeze:

```csharp
// Cache before freeze
Vector3 cachedInertiaTensor = rb.inertiaTensor;
Quaternion cachedInertiaTensorRotation = rb.inertiaTensorRotation;
rb.constraints = RigidbodyConstraints.FreezeAll;

// Restore on unfreeze
rb.constraints = RigidbodyConstraints.None;
rb.inertiaTensor = cachedInertiaTensor;
rb.inertiaTensorRotation = cachedInertiaTensorRotation;
```

In Timber Tycoon the restore lives in `RestoreDriveInertia()`, called when the NPC car transitions from the `Parked` state back to a driving state.

## What didn't work
Relying on Unity to recompute the tensor automatically on unfreeze — it doesn't. The car resumes with a near-zero tensor regardless of physics settings.

## Transferability
Any Rigidbody that toggles `automaticInertiaTensor=false` and then applies freeze constraints — expecting normal rotational physics afterward — is at risk. Common scenarios: parking systems, ragdoll entry/exit, or any "freeze in place while an animation plays" pattern.

## Related
- [Forward axis = -transform.right (Blender FBX quirk)](forward-axis-blender-fbx-quirk.md)
- [Self-collision compound BoxColliders](self-collision-compound-colliders-ignore.md)
