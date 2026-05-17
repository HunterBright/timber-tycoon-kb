---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [blender, origin, pivot, unity, asset-convention]
date: 2026-05-17
status: draft
---

# Asset Origin at Bottom-Center Convention

## When to use
All 3D assets exported from Blender to Unity. Applies to trees, buildings, props, vehicles — every asset that needs to be placed on or snapped to a surface.

## Steps
Per asset type:
- **Trees (adult)**: pivot at base of trunk, NOT center of mesh
- **Buildings**: ground-level center of footprint (Y=0 at floor)
- **Vehicles**: center of wheelbase on ground plane (Y=0 at wheel contact)
- **Props**: exact bottom center (e.g., PelletBag: label-bottom of the bag)

In Blender:
- Object → Set Origin → Origin to 3D Cursor (cursor at desired position), OR
- Object → Set Origin → Origin to Geometry, then manually snap to bottom in Edit Mode

## Why this works
- Ground-snap via raycast needs no offset calculation — pivot touches the surface
- Rotations are natural — object rotates around its base, not its geometric center
- Stacking works — "place asset on surface" = pivot touches surface = correct visual placement

## Trade-offs
Requires deliberate origin placement per asset in Blender (can't just click "Origin to Geometry"). Extra 30 seconds per asset in Blender saves debugging every time placement code is written.

## Variants
Exception: PelletBag / FertilizerBag use exact label-bottom (bottom of print area), not geometric bottom — matters when bags are stacked on shelves.
