---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, physics, rigidbody, tree-cutting, fallen-trunk]
date: 2026-05-28
status: draft
---

# Trunk Fall Physics Config

## When to use
Any game where falling objects (tree trunks, crates, logs) need to tumble briefly on impact and then come to rest without perpetual jitter or expensive physics simulation.

## Steps

**Spawn configuration:**
```csharp
var rb = trunk.GetComponent<Rigidbody>();
rb.mass = 100f;         // heavy = realistic inertia
rb.useGravity = true;
rb.isKinematic = false; // physics-driven during fall

// Apply fall impulse at spawn
rb.AddForce(fallDirection * fallForce, ForceMode.Impulse);

// Start settle capture after brief tumble window
StartCoroutine(StabilizeTrunk(rb, 0.3f));
```

**Settle-and-capture coroutine** (does NOT set isKinematic, does NOT snap position):
```csharp
IEnumerator StabilizeTrunk(Rigidbody rb, float delay) {
    yield return new WaitForSeconds(delay); // brief tumble window (juice)
    // Poll until physics settles the body
    while (rb.velocity.sqrMagnitude > 0.01f || rb.angularVelocity.sqrMagnitude > 0.01f)
        yield return new WaitForFixedUpdate();
    // Capture final pose for save/load — never impose it
    settledPosition = rb.transform.position;
    settledRotation = rb.transform.rotation;
    // PhysicsOptimizer detects the body is still and sleeps it automatically
}
```

**BoxCollider config** (from prefab — do NOT override in code, see [[script-overrides-prefab-inspector-values]]):
- Center: aligned to the mesh bounds center (NOT the pivot — pivot is often at the base of the trunk, not its geometric center)
- Size: matches the mesh bounds extents

Note: a CapsuleCollider rests on its rounded end at an angle on uneven terrain — a BoxCollider lies flat on its face. Center misaligned to the pivot (e.g. using Vector3.zero) makes the trunk float or half-sink.

**Player grace period:** ignore collision between trunk and player for 1s after spawn — prevents trunk from knocking player down at the spawn point.
```csharp
Physics.IgnoreCollision(trunk.GetComponent<Collider>(), playerCollider, true);
StartCoroutine(RestoreCollision(trunk.GetComponent<Collider>(), playerCollider, 1f));
```

## Why this works
Physics settles the trunk naturally — the same way spawned Logs settle in the same project. PhysicsOptimizer detects when the body stops moving and sleeps it (zero CPU cost). The settle coroutine only records the final resting pose for persistence; it never imposes a pose. World feels "settled" without fighting the engine.

## Trade-offs
- Multiple trunks at once: each trunk has its own 0.3s tumble window. For 5 simultaneous cuts, peak is 5 active rigidbodies — acceptable
- isKinematic → kinematic stops force exchange between trunks. Two kinematic trunks don't push each other — acceptable for low-poly style
- 0.3s tumble window: this is for visual juice (brief rolling/tumbling), not for stabilization — settling is physics-driven and completes whenever the engine is satisfied, not on a timer

## Variants
- **No physics fall:** trunk plays a falling animation (no Rigidbody), converts to static collider on landing — simpler, less dynamic
- **Ragdoll trunk:** Rigidbody chain per log segment — cinematic but expensive

See also: [[capsule-collider-direction-axis]], [[script-overrides-prefab-inspector-values]], [[dynamic-rigidbody-no-nonconvex-meshcollider]], [[snap-freeze-instead-of-fixing-physics-cause]]
