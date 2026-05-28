---
type: anti-pattern
project: timber-tycoon
suggested-category: engine/anti-patterns
tags: [unity, physics, rigidbody, debugging, process]
severity: high
date: 2026-05-28
status: draft
---

# ANTI-PATTERN: Snap/Freeze to Mask a Physics Bug Instead of Fixing the Cause

When a physics object won't settle correctly, forcing its position/rotation (snap) or freezing it (isKinematic) hides the symptom and spawns new bugs, instead of fixing why it misbehaves.

## The Trap
An object jitters, sinks, or won't rest. The "quick fix" is: after a delay, set isKinematic=true, zero velocity, raycast down and snap position to the ground, zero the tilt. It looks fixed in one test case, then breaks in others.

## Why It Bites
Each manual override fights the physics engine and assumes a pose the engine never produced:
- A single-point ground raycast snaps a long object's pivot, burying one end and floating the other on sloped/uneven terrain.
- Zeroing tilt assumes the mesh's neutral pose is "lying flat" — often it is the modelled-upright pose, so the object stands up.
- Freezing mid-motion locks in whatever chaotic pose the timer happened to catch.
Every override is a new assumption that fails on the next case. The real cause (spawn overlap, wrong collider, missing sleep registration) stays unfixed underneath.

## Symptoms
- "Fix" works for one tree/object, fails for the next.
- Object rests tilted, one end buried, other end floating.
- Object freezes mid-air or mid-slide.
- Each fix-attempt produces a different new artifact.

## Correct Approach
Find why the engine won't rest it naturally, then remove the overrides:
- Spawn-frame launch → IgnoreCollision against overlapping objects (stump, sapling) BEFORE the first FixedUpdate.
- Never settles / jitters forever → object isn't registered with the sleep optimizer, or collider/damping is wrong for its mass+shape.
- Sinks into ground → collider center misaligned with mesh, or collider missing.
Let physics own the resting pose, exactly like a working reference object in the same project (here: spawned Logs settle under physics, no snap, no forced freeze).

## Detection
If your "settle" code writes transform.position or transform.eulerAngles, or sets isKinematic on a timer, you are probably masking a cause. A settle routine should at most CAPTURE the physics-settled pose (for save/load), not impose one.

## See also
- [[trunk-fall-physics-config]]
- [[dynamic-rigidbody-no-nonconvex-meshcollider]]
- [[read-actual-code-before-hypothesizing]]
