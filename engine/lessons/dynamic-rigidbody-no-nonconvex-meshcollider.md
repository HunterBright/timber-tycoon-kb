---
title: Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- rigidbody
- meshcollider
- collider
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
---

# Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh

## Problem
A falling/moving object that needs a MeshCollider either falls through the world or throws "Non-convex MeshCollider with non-kinematic Rigidbody is no longer supported." Hours lost trying to force mesh-accurate collision on a physics-driven object.

## Root cause
Unity (6000.3, and all versions since 5.0) forbids a non-convex MeshCollider on a non-kinematic Rigidbody - PhysX cannot simulate arbitrary concave shapes in motion. A falling debranched trunk is non-kinematic during its fall, so a non-convex MeshCollider on it is illegal. Convex MeshCollider is allowed, but it is only the convex hull of the mesh - for a cylinder/log it is functionally a fatter, more expensive, less stable primitive.

## Solution
For any object that is non-kinematic at any point (falls, rolls, is pushed):
- Use a primitive collider (Box / Capsule / Sphere), or a convex MeshCollider if the shape genuinely needs it.
- For long lying objects (logs, trunks): BoxCollider beats CapsuleCollider - a capsule rests on its rounded end at an angle on uneven terrain; a box lies flat on its face.
- Mesh-accurate (non-convex) colliders are ONLY for static or kinematic objects (terrain, buildings, slept rigidbodies).

A static reference object can keep a non-convex MeshCollider precisely because it is never non-kinematic. Don't reason "object X uses MeshCollider and works, so my falling object can too" - check whether X is ever non-kinematic. If it sleeps/stays kinematic, it is not a valid analogy.

## What didn't work
Trying to give a falling trunk a non-convex MeshCollider "to match Spruce_Log" - Log works because it is slept (kinematic) almost immediately; the trunk is non-kinematic through its whole fall.

## Transferability
Any Unity project with physics-driven props that "look like they need" an accurate collider. The rule is universal: motion state, not visual fidelity, decides collider type.

## Related
- [[trunk-fall-physics-config]]
- [[capsule-collider-direction-axis]]
- [[collider-distribution-rule]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-1110-meshcollider-niewypukly-z-rigidbody-gubi-kolizje|Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata]] - wspolne: rigidbody, meshcollider, physics
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] - wspolne: rigidbody, collider, physics
- [[20260622-0950-unity-map-boundary-from-bounds|Invisible map boundary from live Renderer.bounds + foot-only gap via IgnoreCollision]] - wspolne: rigidbody, collider, physics
- [[20260710-2115-collider-from-first-meshfilter-antipattern|Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu]] - wspolne: meshcollider, collider, physics
- [[20260714-2220-maxspeed-clamp-is-not-a-speed|maxSpeed to KLAMRA, nie prędkość - pojazd i tak stanie na (napęd / tłumienie)]] - wspolne: rigidbody, physics
- [[20260724-0907-arcade-car-climbs-walls|Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo]] - wspolne: rigidbody, physics
<!-- /POWIAZANE:auto -->
