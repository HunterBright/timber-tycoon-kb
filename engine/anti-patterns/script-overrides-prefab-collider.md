---
name: script-overrides-prefab-collider
description: Code adds CapsuleCollider with microscopic hardcoded values — overrides correct prefab values. Fix: GetComponent first, only AddComponent if absent, never overwrite existing.
metadata:
  type: anti-pattern
  project: timber-tycoon
  suggested-category: engine/anti-patterns
  tags: [unity, prefabs, inspector, scripts, debugging]
  severity: high
  time_lost: 90min (trunk levitation debugging)
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ANTI-PATTERN: Script Overrides Prefab Inspector Values (CapsuleCollider Case)

Merger of sweep2 "CapsuleCollider script override" + #112 (specific trunk physics case). The general rule covers any prefab field; this file documents the CapsuleCollider case that burned 90 minutes.

## The Trap
ChoppableTree spawn code `AddComponent<CapsuleCollider>()` and then hardcodes `radius = 0.0075f`, `height = 0.0024f`. But the trunk prefab already has a correctly-sized CapsuleCollider from the Inspector. Result: two CapsuleColliders on the object, one microscopic — trunk levitates or passes through ground.

```csharp
// ❌ Anti-pattern: hardcodes over prefab values
CapsuleCollider col = trunk.AddComponent<CapsuleCollider>(); // ADDS second collider
col.radius = 0.0075f;  // microscopic — wrong
col.height = 0.0024f;  // microscopic — wrong
```

## Why It Fails
- `AddComponent` doesn't check if a component already exists — creates a duplicate
- Hardcoded values are wrong (developer guessed without checking the prefab)
- Two colliders on one Rigidbody → physics engine gets confused, produces unpredictable results
- Symptoms: trunk spawns and immediately levitates, OR trunk falls through terrain, OR trunk sinks halfway — all caused by the same bug

## Correct Pattern
```csharp
// ✅ Correct: use existing, add only if missing, NEVER override existing values
CapsuleCollider col = trunk.GetComponent<CapsuleCollider>();
if (col == null) {
    col = trunk.AddComponent<CapsuleCollider>();
    // Only set values when CREATING (no prefab values to trust yet)
    col.direction = 0;    // X-axis for lying log (horizontal)
    col.radius = 0.15f;   // correct size for a log
    col.height = 3.5f;    // correct trunk length
    col.center = Vector3.zero;
}
// If col already exists: trust Inspector values — don't touch them
```

**General rule:** if a prefab field is designer-tunable (visible in Inspector), **code must not overwrite it**. The Inspector value is the design intent. Code overwriting it silently discards that intent.

**Detection:** if an Inspector edit to a collider/field has no effect in Play Mode, search for `col.radius =` or the field name + `=` in code — you've found the overwriting code.

## Companion anti-pattern
`script-overrides-prefab-inspector-values.md` (see [[script-overrides-prefab-inspector-values]]) covers the general case for all component types. This file is the concrete CapsuleCollider incident.

## See also
[[trunk-fall-physics-config]], [[capsule-collider-direction-axis]], [[script-overrides-prefab-inspector-values]]
