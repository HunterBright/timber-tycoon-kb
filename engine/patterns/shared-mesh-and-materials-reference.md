---
name: shared-mesh-and-materials-reference
description: Placeholder GameObjects reference the SAME mesh + materials as the final asset — no duplication, no placeholder-vs-final drift
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, prefabs, references, placeholders, debug]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Shared Mesh + Materials Reference Pattern

## When to use
During development when a final prefab isn't ready but the visual representation needs to be in scene (for positioning, testing, gameplay logic). Use placeholder GameObjects that reference the same assets as the future final.

## Steps

**Anti-pattern (don't do this):**
```
PelletRack_Placeholder/
  MeshFilter → placeholder_box.mesh       ← different mesh
  MeshRenderer → Mat_Placeholder_Debug    ← different material
```
When final is ready: manual asset swap needed, visual drift during development, placeholder looks wrong in screenshots/demos.

**Correct pattern:**
```
PelletRack_Placeholder/
  RackVisual/
    MeshFilter → BagRack.fbx (BackPanel + Posts + Shelves)   ← SAME mesh
    MeshRenderer → Mat_BagRack_Frame                         ← SAME material
  StorageRack (component, family=PelletBag, capacity=9)
  SlotAnchor_0_0, SlotAnchor_0_1, ... (positioned manually)
```

**Difference between placeholder and final = only transform, layer, and components — NOT mesh/material assets.**

**Workflow:**
1. Drop `PelletRack_Placeholder` in scene, configure `StorageRack` component
2. Iterate gameplay (storage logic, fill levels) — visual is already correct
3. When final prefab arrives: swap GameObject in scene, copy transform. Runtime behavior unchanged.

**Update propagation:** modify `BagRack.fbx` in Blender → re-import → placeholder updates automatically.

## Why this works
Unity's asset reference system means both placeholder and final share the same mesh/material asset objects. Visual parity is guaranteed. Demo screenshots look polished even before "final" assets exist.

## Trade-offs
- Placeholder still has `_Placeholder` suffix in name — signal it's work-in-progress, even though it looks final
- If placeholder needs to be invisible in-game (debug only), use `HideFlags` or a separate debug-only scene
- SlotAnchor positions may need manual adjustment when final mesh dimensions differ from placeholder

## Variants
- **Fully identical prefab early:** just create the actual prefab immediately and stop thinking of it as "placeholder" — sometimes faster
- **Debug material layer:** give placeholder GameObject a separate layer that's excluded from production builds

See also: [[rack-visual-fill-alignment]]
