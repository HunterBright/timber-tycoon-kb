---
title: Walkable cave from a hollow low-poly rock (don't carve the model)
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- environment
- cave
- meshcollider
- map-boundary
- level-design
- raycast
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Walkable cave from a hollow low-poly rock (don't carve the model)

## When to use
A decorative low-poly rock/boulder/cliff visually suggests a cave, but the player
hits an invisible wall partway in, or there's no floor inside. You want a real
enclosed walkable cave WITHOUT re-modelling in Blender — when the rock is a shell
(hollow underneath the overhang) rather than a solid block.

## Steps
1. **Probe whether the rock is hollow.** From a point at head height inside the
   rock footprint, raycast UP (ceiling), DOWN (is there floor?), and the 4 horizontal
   dirs. Big UP distance + no DOWN hit = hollow shell with headroom = good candidate.
   Exclude the invisible boundary layer from these rays (`mask = ~(1<<boundaryLayer)`).
2. **Find the floor seam height.** Raycast down (terrain layer only) where the cave
   meets existing ground; build the new floor flat at that Y minus ~5 cm (avoids
   z-fighting at the seam; the imperceptible step reads as flush).
3. **Drop an embedded floor slab.** Build a subdivided quad in memory (`new
   UnityEngine.Mesh()`), assign to a scene object's MeshFilter + MeshCollider, REUSE
   the rock's material (triplanar = no UVs needed, floor matches walls). Put it on the
   ground/terrain layer, Cast Shadows Off. Unity embeds the mesh in the scene on save.
4. **Wrap the map boundary as a U-pocket, not a straight wall.** Cut a mouth in the
   perimeter wall at the cave entrance, then add 3 invisible BoxCollider walls (back +
   2 sides) hugging the slab edges. The rock is the visual ceiling/upper walls; the
   invisible pocket is the floor-level containment. Net: cave becomes an enclosed room
   inside the playable area, with one opening back to the map — player can't fall into
   the void behind the rock.
5. **Light it.** Enclosed rock = near-black. Add one dim warm point light.
6. **Verify in two passes.** BoxColliders (boundary) cook in edit mode — raycast them
   immediately to prove mouth-open + chamber-enclosed. An embedded runtime MeshCollider
   does NOT cook for raycasts until the scene is saved/loaded — verify the floor is
   solid only AFTER saving.

## Why this works
Low-poly "rocks" are usually thin shells: the overhang already forms ceiling and
walls with real headroom; only the floor is missing. A flat slab + a boundary pocket
is far cheaper than carving a watertight cave in the model, and reusing the rock's
triplanar material makes the flat floor blend into the faceted walls.

## Trade-offs
- Boundary is now hand-built — any "rebuild the perimeter ring" tool that destroys and
  regenerates the boundary root will wipe the cave pocket (and any other hand-added
  walls). Either stop using that tool or teach it to preserve/reproduce the pocket.
- Floor is flat vs organic walls; fine for a small cave, add vertex noise if it reads
  too clean. Natural rock pillars poking up through the slab actually help.

## Variants
- **Secret exit** instead of a room: keep the mouth as a foot-gate (player passes,
  vehicles blocked via `Physics.IgnoreCollision` on the CharacterController) and let
  the slab lead to an off-map area.
- If the rock is genuinely SOLID (DOWN ray hits immediately, no headroom), this pattern
  doesn't apply — you must carve the model in Blender first.
