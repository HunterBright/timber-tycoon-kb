---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, water, wading, slowdown, gameplay-zone]
date: 2026-05-17
status: draft
---

# WaterZone Gameplay Component

## When to use
Any open-world map with a water body (river, lake, swamp) that the player can optionally cross. Without gameplay modification, water is just a visual element. With WaterZone, it becomes a meaningful strategic option.

## Steps

**WaterZone component:**
```csharp
public class WaterZone : MonoBehaviour {
    [SerializeField] float speedMultiplier = 0.5f;  // 50% movement
    [SerializeField] float wadingDepth = 0.6f;       // meters below water surface
    [SerializeField] float waterSurfaceY;             // world Y of water mesh

    void OnTriggerEnter(Collider other) {
        if (!other.TryGetComponent<PlayerController>(out var pc)) return;
        pc.speedMultiplier *= speedMultiplier;
        // Audio: play splash + start wade loop
        audioManager.PlaySFX(splash);
        audioManager.PlayAmbient(wade);
    }

    void OnTriggerStay(Collider other) {
        if (!other.TryGetComponent<PlayerController>(out var pc)) return;
        // Clamp player Y to wading depth (visual submersion)
        var pos = pc.transform.position;
        pos.y = Mathf.Min(pos.y, waterSurfaceY - wadingDepth);
        pc.transform.position = pos;
    }

    void OnTriggerExit(Collider other) {
        if (!other.TryGetComponent<PlayerController>(out var pc)) return;
        pc.speedMultiplier /= speedMultiplier; // restore
        audioManager.StopAmbient(wade);
        audioManager.PlaySFX(splash);
    }
}
```

**BoxCollider:** covering the river width/depth, marked as `isTrigger = true`. River mesh itself has no collider (can walk through it).

**Wading depth clamp:** `waterSurfaceY - 0.6f` = player submerged to waist. Below-waist submersion is cosmetic only — CharacterController still handles ground movement.

**Strategic choice:** cross river directly (slower, shorter path) vs. go around bridge (longer, full speed). Player chooses based on urgency. Adds map richness.

**Rejected for MVP:**
- Swim system (complex, out of scope)
- "Deep water = death" (too punishing, forgiving design preferred)
- Splash VFX (deferred, see [[vfx-wycofane-decision]])

## Why this works
Simple trigger + speedMultiplier = one component, complete gameplay effect. No specialized movement system needed. The strategic tradeoff (shorter-slower vs. longer-faster) adds depth without complexity.

## Trade-offs
- `speedMultiplier` stack: if player enters multiple WaterZone triggers (river + flood area), multiplier compounds. Add a zone counter: restore fully on last exit, not every exit
- Vehicle in water: vehicle doesn't use `PlayerController.speedMultiplier` — add separate `VehicleWaterZone` handling (engine rev down, speed cap)
- Ripple/displacement VFX: water shader doesn't respond to player — would require GPU displacement or mesh manipulation. Deferred.

## Variants
- **Swimming zone:** deeper water, player switches to swim locomotion. Higher complexity but more immersive for games with large water bodies
- **Current zone:** directional force applied to player in zone (river current pushes downstream). Easy to add: `rb.AddForce(currentDir * currentStrength)` in OnTriggerStay
