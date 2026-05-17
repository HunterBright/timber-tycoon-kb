---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, storage, routing, decoupled, manager]
date: 2026-05-17
status: draft
---

# GLOBAL_ROUTER Pattern (StorageManager.AddToStorage)

## When to use
Any system where multiple producers (machines, pickup zones) output items that need to go into specific storage containers, but producers shouldn't need to know which container exists or where it is.

## Steps

**Caller code (machine output):**
```csharp
// PlankMaker finishes — just announces what was made:
int overflow = StorageManager.Instance.AddToStorage(ProductType.Plank, 4);
if (overflow > 0) Debug.Log($"{overflow} planks couldn't be stored (rack full)");
```

**StorageManager.AddToStorage() internals:**
```csharp
public int AddToStorage(ProductType productType, int count) {
    // 1. Map ProductType → StorageFamily
    var family = StorageFamilyMapping.GetFamily(productType);

    // 2. Find rack via registry
    if (!StorageRackRegistry.Instance.TryGetRackByFamily(family, out var rack))
        return count; // no rack = full overflow

    // 3. Deposit
    return rack.Add(productType, count); // returns remaining overflow
}
```

**StorageFamilyMapping (static class or SO):**
```csharp
public static StorageFamily GetFamily(ProductType t) => t switch {
    ProductType.Log       => StorageFamily.Log,
    ProductType.Stump     => StorageFamily.Stump,
    ProductType.Firewood  => StorageFamily.Firewood,
    ProductType.WoodChips => StorageFamily.ChipBag,
    ProductType.Plank     => StorageFamily.Plank,
    ProductType.Bark      => StorageFamily.Bark,
    ProductType.Pellet    => StorageFamily.PelletBag,
    _                     => StorageFamily.Default
};
```

**Adding new product type:**
1. Add `ProductType.NewType` to enum
2. Add mapping in `StorageFamilyMapping`
3. Add `StorageFamily.NewRack` if needed
4. Place rack in scene with matching family
5. Done — producers already work

**ProductType enum (TT):** Log, Stump, Firewood, WoodChips, Plank, Bark, Pellet, Fertilizer, Furniture

## Why this works
Router is a single indirection layer between producers and storage. Producers only need to know their ProductType — routing is not their concern. Adding a new rack type = data change, not a code change across all producers.

## Trade-offs
- ProductType → StorageFamily mapping: if two ProductTypes share a rack family, both go to the same rack (may or may not be intended — check at design time)
- Overflow return value: callers should log or notify player when items can't be stored. Silently dropping overflow = invisible data loss
- Multiple racks per family (overflow routing): extends to `List<StorageRack>` per family — try first until capacity found

See also: [[storage-rack-registry-auto-register]], [[dictionary-warehouse-registry]], [[storage-rack-family-system]]
