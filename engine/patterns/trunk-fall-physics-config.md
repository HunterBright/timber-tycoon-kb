---
name: trunk-fall-physics-config
description: Fallen trunk Rigidbody config — mass=80, isKinematic=false at spawn, auto-stabilize to kinematic after 1.5s to prevent jitter
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, physics, rigidbody, tree-cutting, fallen-trunk]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Trunk Fall Physics Config

## When to use
Any game where falling objects (tree trunks, crates, logs) need to tumble briefly on impact and then come to rest without perpetual jitter or expensive physics simulation.

## Steps

**Spawn configuration:**
```csharp
var rb = trunk.GetComponent<Rigidbody>();
rb.mass = 80f;          // heavy = realistic inertia
rb.useGravity = true;
rb.isKinematic = false; // physics-driven during fall

// Apply fall impulse at spawn
rb.AddForce(fallDirection * fallForce, ForceMode.Impulse);

// Stabilize after delay
StartCoroutine(StabilizeTrunk(rb, 1.5f));
```

**Auto-stabilize coroutine:**
```csharp
IEnumerator StabilizeTrunk(Rigidbody rb, float delay) {
    yield return new WaitForSeconds(delay);
    rb.isKinematic = true; // freeze in place
    rb.velocity = Vector3.zero;
    rb.angularVelocity = Vector3.zero;
}
```

**CapsuleCollider config** (from prefab — do NOT override in code, see [[script-overrides-prefab-collider]]):
- Direction: 0 (X-axis, along trunk length)
- Radius: 0.15
- Height: 3.5

**Player grace period:** ignore collision between trunk and player for 1s after spawn — prevents trunk from knocking player down at the spawn point.
```csharp
Physics.IgnoreCollision(trunk.GetComponent<Collider>(), playerCollider, true);
StartCoroutine(RestoreCollision(trunk.GetComponent<Collider>(), playerCollider, 1f));
```

## Why this works
`isKinematic = false` for 1.5s = trunk falls and rolls naturally. After 1.5s switch to `isKinematic = true` = frozen in place, zero physics cost (static rigidbody). World feels "settled."

## Trade-offs
- Multiple trunks at once: each trunk has its own 1.5s timer. For 5 simultaneous cuts, peak is 5 active rigidbodies — acceptable
- isKinematic → kinematic stops force exchange between trunks. Two kinematic trunks don't push each other — acceptable for low-poly style
- 1.5s stabilization: trunk can still slide during this window. Adjust to 0.8s for fast-paced gameplay where player needs to collect quickly

## Variants
- **No physics fall:** trunk plays a falling animation (no Rigidbody), converts to static collider on landing — simpler, less dynamic
- **Ragdoll trunk:** Rigidbody chain per log segment — cinematic but expensive

See also: [[capsule-collider-direction-axis]], [[script-overrides-prefab-collider]]
