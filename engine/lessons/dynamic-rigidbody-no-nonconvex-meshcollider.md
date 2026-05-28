---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, physics, rigidbody, meshcollider, collider]
severity: high
date: 2026-05-28
status: draft
---

# Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh

## Problem
A falling/moving object that needs a MeshCollider either falls through the world or throws "Non-convex MeshCollider with non-kinematic Rigidbody is no longer supported." Hours lost trying to force mesh-accurate collision on a physics-driven object.

## Root cause
Unity (6000.3, and all versions since 5.0) forbids a non-convex MeshCollider on a non-kinematic Rigidbody — PhysX cannot simulate arbitrary concave shapes in motion. A falling debranched trunk is non-kinematic during its fall, so a non-convex MeshCollider on it is illegal. Convex MeshCollider is allowed, but it is only the convex hull of the mesh — for a cylinder/log it is functionally a fatter, more expensive, less stable primitive.

## Solution
For any object that is non-kinematic at any point (falls, rolls, is pushed):
- Use a primitive collider (Box / Capsule / Sphere), or a convex MeshCollider if the shape genuinely needs it.
- For long lying objects (logs, trunks): BoxCollider beats CapsuleCollider — a capsule rests on its rounded end at an angle on uneven terrain; a box lies flat on its face.
- Mesh-accurate (non-convex) colliders are ONLY for static or kinematic objects (terrain, buildings, slept rigidbodies).

A static reference object can keep a non-convex MeshCollider precisely because it is never non-kinematic. Don't reason "object X uses MeshCollider and works, so my falling object can too" — check whether X is ever non-kinematic. If it sleeps/stays kinematic, it is not a valid analogy.

## What didn't work
Trying to give a falling trunk a non-convex MeshCollider "to match Spruce_Log" — Log works because it is slept (kinematic) almost immediately; the trunk is non-kinematic through its whole fall.

## Transferability
Any Unity project with physics-driven props that "look like they need" an accurate collider. The rule is universal: motion state, not visual fidelity, decides collider type.

## Related
- [[trunk-fall-physics-config]]
- [[capsule-collider-direction-axis]]
- [[collider-distribution-rule]]
