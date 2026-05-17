---
name: collider-distribution-rule
description: Selective collider rule — BoxCollider on Foundation/Walls/Pillars (solid), NO collider on Beams/Roofs (visual-only) — reduces physics scene cost
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, colliders, performance, architecture, level-design]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Collider Distribution Rule (Architecture)

## When to use
Any architectural structure (building, machine housing, bridge) where the designer controls which elements the player can collide with. Default behavior (collider on everything) bloats the physics scene.

## Steps

**Collider assignment by element type:**

| Element | Collider | Reason |
|---------|----------|--------|
| Foundation | BoxCollider | Player walks on it, can't fall through |
| Wall | BoxCollider | Player can't pass through |
| Pillar | BoxCollider | Player can't clip through |
| Beam (overhead) | **NONE** | Too high to interact with, no gameplay benefit |
| Roof | **NONE** | Player can't access roof (no ladder system) |
| Doorframe | **NONE** | Covered by surrounding walls, gap handled by waypoints |

**Why beams have no collider:**
- Player can walk under them (high clearance)
- Removing collider: simpler player navigation (no head-bump on low frames), fewer raycast hits, smaller PhysX scene
- Beams are visual information only — removing them from physics doesn't change any gameplay

**Setup script:**
```csharp
// Automatically add BoxColliders to walls/pillars/foundation:
foreach (var t in building.GetComponentsInChildren<Transform>()) {
    if (t.name.StartsWith("Wall_") || t.name.StartsWith("Pillar_") || t.name == "Foundation") {
        if (t.GetComponent<Collider>() == null)
            t.gameObject.AddComponent<BoxCollider>();
    }
    // Beams, Roofs: no action (leave collider-free)
}
```

**Adjust BoxCollider size** to match mesh bounds — Unity auto-sizes on Add, but verify no overshoots.

## Why this works
Physics engine iterates all colliders in the broad-phase per frame. Fewer colliders = faster broad-phase = more physics budget for dynamic objects. Visual-only elements (beams, roofs) have zero gameplay impact when excluded.

## Trade-offs
- Open roofs: if game design changes to allow rooftop access (ladder, parkour), roofs need colliders added retroactively
- Beam head-bumping: in tight indoor spaces, beams at ~2m height can block player movement. If beams are low, add BoxCollider with `isTrigger = true` for soft height-limit (slow player, not block)
- Missing collider check: if player falls through world, the first debug step is verifying Foundation has a BoxCollider

## Variants
- **MeshCollider for complex shapes:** for organic rock walls or curved foundations, MeshCollider + convex=false — more accurate but heavier
- **Trigger zones:** doorways with Trigger colliders for zone detection (entering sawmill triggers ambient change, etc.)

See also: [[architectural-naming-convention]]
