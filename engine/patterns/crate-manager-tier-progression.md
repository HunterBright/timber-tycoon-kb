---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, crate, tier, upgrade, inventory]
date: 2026-05-17
status: draft
---

# CrateManager Tier Progression

## When to use
Games where the player has a carry capacity that upgrades over time (crate, backpack, cargo hold). Upgrades should be event-driven — no per-upgrade code needed once the tier system exists.

## Steps

**CrateTier enum and slot count:**
```csharp
public enum CrateTier { Small_10 = 10, Medium_15 = 15, Large_25 = 25 }
```

**CrateManager singleton:**
```csharp
public class CrateManager : MonoBehaviour, ISaveable {
    public CrateTier CurrentTier { get; private set; } = CrateTier.Small_10;
    public int SlotCount => (int)CurrentTier;

    List<CrateSlot> slots = new(); // (ProductType type, int amount) per slot

    void OnEnable()  => onUpgradePurchased.Register(OnUpgradePurchased);
    void OnDisable() => onUpgradePurchased.Unregister(OnUpgradePurchased);

    void OnUpgradePurchased(string id) {
        if (id == "CrateMedium") CurrentTier = CrateTier.Medium_15;
        if (id == "CrateLarge")  CurrentTier = CrateTier.Large_25;
        OnTierUpgraded?.Invoke(CurrentTier);
    }

    public bool LoadItem(ProductType type, int amount) {
        if (slots.Count >= SlotCount) return false; // full
        slots.Add(new CrateSlot(type, amount));
        return true;
    }

    public List<CrateSlot> UnloadAll() {
        var contents = new List<CrateSlot>(slots);
        slots.Clear();
        return contents;
    }
}
```

**CrateSlot struct:**
```csharp
[Serializable]
public struct CrateSlot {
    public ProductType type;
    public int amount;
}
```

**Visual:** car flatbed shows slot markers based on tier. `OnTierUpgraded` event → flatbed visual updates slot count indicator. Each tier uses a different FBX (Small/Medium/Large) in `CarryCrate_v2`.

**ISaveable:** serialize `CurrentTier` + slot contents array.

**Tier as progression moment:** each crate upgrade is visible to the player (more slot markers, different model). Tutorial points to first crate upgrade ~10 minutes in.

## Why this works
Tier upgrade via event subscription = CrateManager is self-updating. Upgrade shop doesn't call CrateManager directly — raises event, CrateManager responds. `enum CrateTier { Small_10 = 10, Medium_15 = 15 }` embeds the slot count in the enum value itself.

## Trade-offs
- Slot count = `(int)CurrentTier`: works cleanly only if tier names follow `Name_N` pattern. If design changes slot counts mid-project, this coupling breaks — consider a separate `Dictionary<CrateTier, int>` lookup
- Unload all at once: player can't selectively leave items in crate. For more control, add `Unload(int slotIndex)` method
- Multiple crates: currently assumes 1 crate per player. For co-op or vehicle-mounted crates, scope CrateManager per vehicle/player entity

See also: [[storage-activation-gating-upgrade]], [[isaveable-contract]]
