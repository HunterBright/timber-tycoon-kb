---
type: anti-pattern
project: timber-tycoon
suggested-category: engine/anti-patterns
tags: [unity, prefabs, inspector, scripts, debugging]
date: 2026-05-17
status: draft
---

# ANTI-PATTERN: Script Overrides Prefab Inspector Values

Code overwrites Inspector-set values silently — designer edits have no effect in Play Mode.

## The Trap

Setting values in the Inspector on a prefab, then having code in `Awake`, `Start`, or a spawn method overwrite those values with hardcoded constants. The designer edits the Inspector field and nothing changes in Play Mode. `AddComponent` doesn't check for existing components — creates a duplicate.

## Why It Fails

Code runs AFTER Inspector-set values are loaded. The hardcoded override wins every time. The designer has no way to know from the Inspector that their edits are being silently discarded.

## Symptoms

- Inspector edit produces no in-game change
- Value in Play Mode differs from value in Inspector
- Two components of the same type on one object (duplicate from `AddComponent`)
- Designer spends 20+ minutes on each encounter until the code override is found

## Correct Pattern

```csharp
// ✅ Correct: use existing, add only if missing, NEVER override existing values
CapsuleCollider col = trunk.GetComponent<CapsuleCollider>();
if (col == null) {
    col = trunk.AddComponent<CapsuleCollider>();
    // Only set values when CREATING — no prefab values to trust yet
    col.direction = 0;    // X-axis for lying log (horizontal)
    col.radius = 0.15f;
    col.height = 3.5f;
    col.center = Vector3.zero;
}
// If col already exists: trust Inspector values — don't touch them
```

**General rule:** if a prefab field is designer-tunable (visible in Inspector), code must not overwrite it. The Inspector value is the design intent.

**Detection:** if an Inspector edit has no effect in Play Mode, `grep -r "fieldName" Assets/ --include="*.cs"` to find the overwriting code.

**Fix options:**
- **Fix 1 (preferred):** Remove the hardcoded override. Trust the Inspector value. The prefab IS the source of truth.
- **Fix 2 (when value is mathematical/derived):** Read from a ScriptableObject or constant. Make it explicit that code controls this value — don't put it in an Inspector field that looks editable.

## Case 1: CapsuleCollider (trunk physics, 90 min lost)

ChoppableTree spawn code called `AddComponent<CapsuleCollider>()` and hardcoded microscopic values. The trunk prefab already had a correctly-sized CapsuleCollider from the Inspector. Result: two CapsuleColliders on the object — trunk levitated or fell through ground.

```csharp
// ❌ Anti-pattern
CapsuleCollider col = trunk.AddComponent<CapsuleCollider>(); // ADDS second collider
col.radius = 0.0075f;  // microscopic — wrong
col.height = 0.0024f;  // microscopic — wrong
```

Symptoms: trunk spawns and immediately levitates, OR falls through terrain, OR sinks halfway — same root cause: two colliders, one microscopic.

## Case 2: DirtClump (radius/scale)

`DirtClump.Awake()` set `radius = 0.05f` overriding the Inspector value. Designer set it to `0.5f` in the prefab — had no effect. "I set it to 0.5 but in Play Mode it's 0.05."

**Resolution (2026-05-29):** the radius mismatch was eliminated by dropping the runtime-forced SphereCollider entirely. `CreateClump()` previously hardcoded `sc.radius = 0.14f` at spawn, overriding the prefab's `0.2`. The fix switches to a **convex MeshCollider derived from the clump's own mesh** (destroy any existing collider, `AddComponent<MeshCollider>()`, assign `MeshFilter.sharedMesh`, `convex = true`). This removes the whole prefab-vs-code radius argument: there is no `radius` field to override, the collider shape IS the mesh. The lesson generalizes — when code and prefab keep fighting over a primitive collider's numeric size, the cleaner fix is often to derive the collider from geometry rather than to argue about which number wins. See [[mesh-collider-convex-for-clickable-minigame-objects]].

## See also

[[trunk-fall-physics-config]], [[capsule-collider-direction-axis]]
