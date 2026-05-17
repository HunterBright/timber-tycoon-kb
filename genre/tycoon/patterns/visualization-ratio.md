---
name: visualization-ratio-inventory-to-visual
description: 10:1 inventory-to-visual ratio for logs, 20:1 for planks, 5:1 for pellet bags. FloorToInt(currentAmount / ratio) drives slot activation — looks full without rendering each unit.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, visualization, inventory, ratio, scale]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Visualization Ratio (Inventory to Visual Stack)

## When to use
Tycoon games where racks/storage can hold dozens or hundreds of items. Rendering one mesh per inventory unit would exceed vertex budget immediately. Use this pattern whenever inventory count exceeds 10 units and you need a visual representation.

## Steps

**Ratios per material type (store in StorageFamilySO):**

| Family | Ratio | Meaning |
|--------|-------|---------|
| Log | 10:1 | 10 logs = 1 visual stack of 5-6 logs |
| Plank | 20:1 | 20 planks = 1 visual pallet |
| Pellet Bag | 5:1 | 5 bags = 1 visual stack |
| Bark/Sawdust | 50:1 | 50 units = 1 loose pile shape |

**Visual update logic:**
```csharp
public void RefreshVisuals(int currentAmount) {
    int visualStacks = Mathf.FloorToInt(currentAmount / (float)visualizationRatio);
    visualStacks = Mathf.Clamp(visualStacks, 0, slotVisuals.Length);

    for (int i = 0; i < slotVisuals.Length; i++)
        slotVisuals[i].SetActive(i < visualStacks);
}
```

**Slot setup:**
- Slot transforms placed in scene (or runtime-positioned relative to rack)
- Each slot = pre-positioned empty + instantiated prefab (log, plank bundle, bag)
- Slots fill bottom-to-top (or front-to-back) for natural stacking appearance

**When to call RefreshVisuals:**
- On Add() to rack
- On Remove() from rack
- On load (saved `currentAmount` → immediate refresh in Start)

**Design note:** Players won't count. "Looks full" and "looks half-empty" are the only states that matter visually. Ratio precision is irrelevant — visual impression is the goal.

## Why this works
`FloorToInt(currentAmount / ratio)` is O(1). No loop over inventory, no mesh generation. Slot activation (`SetActive`) is cheap. 10:1 ratio for logs means a rack of 60 logs shows 6 visual stacks — full-looking without 60 log meshes.

## Trade-offs
- Imprecise: 1-9 logs looks identical to 0 logs (no visual slots active). Add a "not empty" indicator (single log visible from 1+) if player needs to distinguish empty from "barely any".
- Large ratios (50:1 bark) mean 49 units visually invisible. Acceptable for bulk materials player doesn't inspect closely. For player-facing quantities, keep ratio ≤10.
- Slot count limits visual maximum: if `maxSlots = 6` and ratio is 10:1, visuals max out at 60 units even if rack holds 100. Match `maxSlots × ratio ≈ maxCapacity`.

## Variants
- **Smooth fill:** instead of discrete slot activation, scale a single fill mesh from 0→1 based on fullness. Looks smoother but harder to match real rack geometry.
- **Fill with LOD:** slots near camera render full mesh; distant slots render billboard or lower-poly proxy.

See also: [[storage-rack-family-system]], [[rack-visual-fill-alignment]], [[dictionary-warehouse-registry]]
