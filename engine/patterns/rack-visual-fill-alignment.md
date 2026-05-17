---
name: rack-visual-fill-alignment
description: Storage rack fill visualization via child fillLevel transforms aligned to shelf Y positions — bags rest on shelves deterministically
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, prefabs, racks, fill-visualization, alignment]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Rack Visual Fill Alignment Pattern

## When to use
3D storage racks where product items should visually occupy shelf positions. Without pre-positioned transforms, items float in air or clip through shelves.

## Steps

**Rack model has fixed shelf Y positions (from Blender):**
- Shelf_1: Y = 0.40
- Shelf_2: Y = 1.00
- Shelf_3: Y = 1.60

**StorageRack component has fillLevel transforms:**
```csharp
[SerializeField] Transform fillLevel1; // Y = 0.40 (matches Shelf_1)
[SerializeField] Transform fillLevel2; // Y = 1.00 (matches Shelf_2)
[SerializeField] Transform fillLevel3; // Y = 1.60 (matches Shelf_3)
```

**At runtime, instantiate visual prefabs as children of fillLevel transforms:**
```csharp
// Fill shelf 1 (3 bags on bottom shelf)
for (int i = 0; i < 3; i++) {
    var bag = Instantiate(bagPrefab, fillLevel1);
    bag.localPosition = new Vector3(slotOffsets[i], 0f, 0f);
}
```

**Capacity logic:**
- `maxCapacity = 9` (3 shelves × 3 bags each)
- `fillLevel1` = first 3 items
- `fillLevel2` = items 4–6
- `fillLevel3` = items 7–9
- StorageRack activates/deactivates fillLevel groups based on current count

**Per-product visual definition:** `StorageFamilySO` specifies which prefab to instantiate for each product type (BagOfPellet, Firewood_Basic, Plank_Spruce, etc.)

**Alignment workflow:**
1. Position fillLevel transforms in scene to match Blender shelf Y values
2. Instantiate products as children — they inherit the shelf Y
3. Offset products along X/Z for even spacing within a shelf

## Why this works
Pre-positioned transforms as parents guarantee products land at the correct world height. No raycasting, no physics-based placement, no drift. Fully deterministic — same shelf, same position, every time.

## Trade-offs
- Shelf Y values are hardcoded to match specific model — if model changes, fillLevel transforms must be repositioned
- bake_space_transform bug (see [[bake-space-transform-linked-duplicates-rotation-bug]]): empty node positions may need remapping (X,Y,Z)→(X,Z,-Y) in setup scripts
- Item spacing (X offset) is manually configured per rack type — no automatic spacing algorithm

## Variants
- **Dynamic slot grid:** calculate slot positions procedurally from rack dimensions + item count — more flexible, more code
- **Physics-placed:** drop items with gravity, let them settle on shelves — looks great, unpredictable and expensive
