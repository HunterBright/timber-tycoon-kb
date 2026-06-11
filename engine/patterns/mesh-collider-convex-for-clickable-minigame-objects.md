---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, collider, meshcollider, raycast, minigame, prefab]
date: 2026-05-29
status: validated
---

# Convex MeshCollider for Irregular Clickable Objects

## Problem
A clickable minigame object has an irregular/lumpy visible mesh but a primitive collider (e.g. a SphereCollider). The hitbox no longer matches the silhouette — the player clicks visible geometry and misses, or clicks empty space and hits. Hardcoding the primitive's radius (`sc.radius = 0.14f`) also fights whatever the prefab author set in the Inspector.

## Root cause
A primitive collider approximates the mesh with a fixed shape. Once the mesh becomes irregular (a flattened, lumpy dirt clod instead of a ball), the sphere either over- or under-covers it. The fix is to make the collider BE the mesh shape.

## Solution
Swap the primitive for a **convex MeshCollider derived from the object's own mesh**, built at spawn time:

```csharp
// Remove any existing collider (e.g. the prefab's SphereCollider) so we
// don't end up with two colliders fighting the raycast.
foreach (Collider oldCol in clump.GetComponentsInChildren<Collider>())
    Destroy(oldCol);

// Convex MeshCollider matching the actual mesh — placed on the TAG-bearing root
// so the click raycast's CompareTag + membership check still resolve.
MeshFilter clumpMesh = clump.GetComponentInChildren<MeshFilter>();
MeshCollider meshCol = clump.AddComponent<MeshCollider>();
if (clumpMesh != null) meshCol.sharedMesh = clumpMesh.sharedMesh;
meshCol.convex   = true;
meshCol.isTrigger = false;
```

Three non-obvious requirements:
1. **Place the collider on the tag-bearing ROOT object.** The click handler resolves the hit via `hit.collider.gameObject.CompareTag("DirtClump")` and a `spawnedClumps.Contains(hitObj)` membership check. If the MeshCollider lands on an untagged child mesh, `hitObj` is the child → both checks fail → the object is unclickable. Add the collider to the same GameObject that carries the tag and is tracked in the spawn list, and assign the mesh from `GetComponentInChildren<MeshFilter>()`.
2. **Destroy any pre-existing collider first.** `AddComponent` never replaces — without the `Destroy` loop you end up with the prefab's SphereCollider AND the new MeshCollider, and the raycast can hit the stale one. (See [[script-overrides-prefab-inspector-values]] for the duplicate-collider trap.)
3. **`convex = true` is mandatory here.** Raycast picking needs it, and the clump carries a kinematic Rigidbody — a non-convex MeshCollider is only legal on static/kinematic bodies, and convex is the safe choice for anything that might ever move. See [[dynamic-rigidbody-no-nonconvex-meshcollider]].

`isTrigger = false` because the picking raycast must hit a solid collider.

## What didn't work
Keeping the SphereCollider and hardcoding `radius = 0.14f` at spawn — it both mismatched the irregular mesh AND silently overrode the prefab's own `radius` value (the prefab had `0.2`). The MeshCollider approach deletes the whole prefab-vs-code radius argument: there is no radius to mismatch, the shape is the mesh.

## Transferability
Any Unity project with click/tap-to-collect or click-to-interact objects whose mesh is irregular (rocks, debris, ore chunks, collectibles). The rule: collider on the tag-bearing root, derived from the mesh, convex, no leftover primitive.

## Related
- [[dynamic-rigidbody-no-nonconvex-meshcollider]]
- [[script-overrides-prefab-inspector-values]]
- [[collider-distribution-rule]]
- [[diegetic-3d-button-raycast]]
