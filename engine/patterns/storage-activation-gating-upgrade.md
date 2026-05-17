---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, storage, upgrade, gating, progression]
date: 2026-05-17
status: draft
---

# Storage Activation Gating via Upgrade Purchase

## When to use
Games with progressive content unlocking where storage types map to gameplay progression stages. Player shouldn't have access to pellet storage before they've unlocked the pelletizer.

## Steps

**StorageRack — default inactive:**
```csharp
public class StorageRack : MonoBehaviour {
    [SerializeField] bool isActive = false; // starts locked
    [SerializeField] string unlockId;       // matches upgrade ID in SawmillManager

    void OnEnable() {
        StorageRackRegistry.Instance.Register(family, this);
        onUpgradePurchased.Register(OnUpgradePurchased);
    }

    void OnDisable() {
        StorageRackRegistry.Instance.Unregister(family);
        onUpgradePurchased.Unregister(OnUpgradePurchased);
    }

    void OnUpgradePurchased(string id) {
        if (id == unlockId) Activate();
    }

    void Activate() {
        isActive = true;
        // Update visual: swap from placeholder/transparent to normal material
        rackVisual.SetActive(true);
        placeholderVisual.SetActive(false);
    }

    public bool CanAccept(ProductType type) => isActive; // check before Add()
}
```

**Active on Day 1 (always active = true):**
- LogRack
- StumpRack
- FirewoodRack (Basic)

**Locked initially, unlock via upgrade:**

| Rack | Unlock tier | Gameplay requirement |
|------|-------------|---------------------|
| PlankRack | PlankMaker unlock | Buy PlankMaker machine |
| BarkRack | PlankMaker unlock | Same (bark is byproduct) |
| ChipBagRack | Shredder unlock | Buy Shredder machine |
| PelletBagRack | Pelletizer unlock | Buy Pelletizer machine |

**Visual state:** inactive rack = transparent gray material (or invisible scaffold). Active = full rack model. Tutorial draws player's attention to next unlock.

## Why this works
Player unlocks storage types alongside the machines that produce for those types. Prevents "what is this rack for?" confusion. Upgrade purchase event propagates to racks automatically — no SawmillManager edit needed per new rack type.

## Trade-offs
- `unlockId` is a string: typo-proof via constants or an enum-to-string conversion
- Visual swap: must switch materials in code or have separate child GameObjects (placeholder vs. active mesh) — keep both children in prefab, toggle active state on unlock
- Day 1 racks: still use the same system, just `isActive = true` as default value in Inspector

## Variants
- **Prerequisite chain:** rack A requires rack B to be unlocked first — add `requiredUnlockId` field
- **Capacity upgrade:** same pattern for upgrading rack capacity (10 → 20 → 30 slots) via unlock events

See also: [[storage-rack-registry-auto-register]], [[global-router-storage-pattern]], [[reputation-levels-data-driven]]
