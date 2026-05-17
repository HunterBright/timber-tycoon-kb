---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, storage, family, capacity, racks, sawmill]
date: 2026-05-17
status: draft
---

# StorageRack Family System

## When to use
Tycoon games with multiple rack/storage types that hold different material categories. Use when you need data-driven storage — adding a new material type should require zero code changes, only a new enum value and rack Inspector config.

## Steps

**StorageFamily enum:**
```csharp
public enum StorageFamily {
    Log, Plank, Bark, Firewood, WoodChips,
    PelletBag, FertilizerBag, Furniture, Stump
}
```

**StorageRack component fields:**
```csharp
public class StorageRack : MonoBehaviour {
    [Header("Config")]
    public StorageFamily family;        // designer-set in Inspector
    public int maxCapacity = 60;        // total items
    public int maxDistinctStacks = 4;   // species variety cap
    public bool isActive = true;        // gating (see activation pattern)

    [Header("Runtime")]
    public int currentAmount;           // sum across all species
}
```

**maxDistinctStacks values by family:**
- `Bark`: 1 (only WoodSpecies.None — bark is species-agnostic)
- `Plank`: 3 (3 species variety realistic for single rack)
- `Log`: 4 (handles all 4 active tree species)
- `PelletBag` / `FertilizerBag`: 2 (typically same recipe)

**Auto-registration via OnEnable** (see [[storage-rack-registry-auto-register]]):
```csharp
void OnEnable() => StorageRackRegistry.Instance.Register(this);
void OnDisable() => StorageRackRegistry.Instance.Unregister(this);
```

**Usage — reading via StorageManager facade:**
```csharp
// Caller side:
StorageManager.Instance.Add(StorageFamily.Log, WoodSpecies.Spruce, 3);
StorageManager.Instance.Get(StorageFamily.Plank, WoodSpecies.Oak);
// StorageManager routes to appropriate rack via StorageRackRegistry.TryGetRackByFamily()
```

## Why this works
Storage architecture is fully data-driven. Adding a new family (e.g., "Toy") = add one enum value, configure racks in Inspector. Zero code changes. `maxDistinctStacks` enforces species realism without hard-coded species checks — a Log rack holding 4 species feels natural, a Bark rack holding 4 species would not.

## Trade-offs
- Single rack per family: if two racks exist with the same family, registry uses first-registered. For multiple racks same family, extend registry to return `List<StorageRack>` and let StorageManager select least-full.
- `isActive` gating: deactivated racks can't receive items. Callers must handle `Add()` returning false (no space or inactive). See [[storage-activation-gating-upgrade]].
- Species cap enforcement: `maxDistinctStacks` checked before Add. If rack already has N species, reject new species (not new units of existing species). Prevents rack bloat.

## Variants
- **Multiple racks per family:** StorageRackRegistry returns `List<StorageRack>` → StorageManager fills in order (primary → overflow). Used when one rack capacity isn't enough.
- **Generic item rack:** family-less rack with `List<ItemSO> acceptedItems`. More flexible but loses the data-driven simplicity of enum matching.

See also: [[storage-rack-registry-auto-register]], [[global-router-storage-pattern]], [[storage-activation-gating-upgrade]], [[rack-visual-fill-alignment]]
