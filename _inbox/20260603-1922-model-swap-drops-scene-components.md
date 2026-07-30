---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, fbx, model-swap, colliders, pickup, interactable, import-scale, mesh-bounds]
severity: medium
time_lost: "~1 session of back-and-forth"
date: 2026-06-03
status: draft
applies_to: [unity]
---

# Swapping an FBX-instance GameObject silently drops its scene-side functionality

## Problem
Replaced an old shovel model with a new one (`Shovel_New.fbx`) by re-pointing the
viewmodel prefab field and swapping the world-pickup's meshes. Result: the pickup
became non-functional — pressing E did nothing and the shovel-unlock quest could not
advance. Separately, the new model imported ~60× too small (~1 cm instead of ~1 m),
so it was nearly invisible in both the viewmodel and the world, even though the
in-hand scale value (1.25) "looked tuned".

## Root cause
Two independent issues, both invisible without measurement:
1. **A model file carries only geometry + materials.** Interaction colliders,
   Rigidbody, the GameObject's layer, and Inspector-set component fields
   (e.g. an `Interactable` subclass's name / pickup flags / weight) are scene-side
   data. Re-instantiating from the new FBX over the old object drops ALL of it. The
   working sibling pickup (the axe) had per-child convex **trigger** MeshColliders on
   the "Interactable" layer + a pickup script with specific field values — none of
   which transfer with a mesh swap.
2. **Wrong FBX export scale.** The model was exported with the wrong scale option, so
   it imported at mm size. `shovelScale = 1.25` on a 1 cm model is still ~1 cm — the
   "tuned" value masked the real problem.

## Solution
- **Replicate from a working sibling, don't reconstruct from memory.** Dump the
  working object's exact setup (collider type / isTrigger / layer / size, Rigidbody
  presence + settings, every serialized field) and reproduce it on the new object.
  Here: convex trigger MeshCollider per child mesh, child + root on the Interactable
  layer, no Rigidbody (the template had none), repaired pickup fields.
- **The interaction raycast finds the script via `GetComponentInParent`**, so the
  collider can sit on a child mesh while the script stays on the root — but the
  collider's GameObject must be on the layer the raycast mask includes.
- **Verify scale by measuring, not eyeballing.** Instantiate the FBX at scale 1 and
  read combined `Renderer.bounds` (or `mesh.bounds`) in metres; compare to a
  known-good reference model. Fix at the source (re-export with the project's standard
  FBX scale settings) or, as a band-aid, the importer's Scale Factor.
- **Quest/flag links survived** because the quest advanced via an event
  (`OnInteract → CollectX()`), not a GameObject reference — so once the collider was
  restored, the quest worked again. Check whether your "broken" link is actually a
  reference, or just a missing trigger upstream.

## What didn't work
- Trusting a transform value ("scale is 1.25, so it's fine") instead of measuring real
  world size — the model was 60× too small underneath.
- Assuming the swap was clean because materials were correct (they were embedded in
  the FBX and transferred fine) — colliders/rigidbody/layer/fields did not.

## Transferability
Applies to ANY Unity project where 3D models double as interactable/physics objects
(pickups, tools, props, machines). The principle "a model file is geometry+materials;
everything that makes it *behave* lives in the scene" and the habit of measuring
imported size against a reference are engine-level, genre-independent.

## Related
- FBX Export Standard Settings (existing KB entry — the correct export scale options)
