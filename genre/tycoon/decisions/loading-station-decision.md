---
name: loading-station-vs-walking-decision
description: Loading crate — walk to each rack manually early game (engagement), centralized LoadingStation as lvl 8 upgrade (QoL reward). Manual walk is gameplay; convenience is earned.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, qol, progression, loading]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# LoadingStation Decision — Manual Walk Early, Station Late

## Decision

**Chosen:** Walk to racks manually in early game (Option A). LoadingStation unlocked as Level 8 upgrade (Option B). Both, staged by progression.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| A | Player walks to each rack manually to load crate | Early game |
| B | Centralized LoadingStation: one interaction to load crate | Late game upgrade (Lvl 8) |

## Reasoning

**Option A alone fails long-term:** Late game with 50+ products to load = 50 walks = tedious. Core loop becomes friction, not fun.

**Option B alone skips engagement:** If LoadingStation exists from day 1, the player never navigates the sawmill. Loses "sense of place" — walking past your machines and racks is part of feeling like an owner.

**Staged solution gives best of both worlds:**
- Early game: manual walking is intentional. Player learns the layout, sees each rack, understands capacity.
- Level 8: LoadingStation is a meaningful QoL reward. Player has earned the convenience.
- Player who upgrades feels: "I remember when I had to walk to every rack. Now I just tap the station."

## Implementation

```csharp
public class LoadingStation : MonoBehaviour, IInteractable {
    public bool isUnlocked = false; // default: false

    void OnEnable() {
        UpgradeManager.OnUpgradePurchased += OnUpgradePurchased;
    }

    void OnUpgradePurchased(string id) {
        if (id == "loading_station") isUnlocked = true;
    }

    public void Interact(PlayerController player) {
        if (!isUnlocked) { /* show "not yet unlocked" tooltip */ return; }
        // Open panel: player selects N of product X → system pulls from racks
    }
}
```

## Lesson

Quality-of-life upgrades are most valued when the player has experienced the friction they eliminate. Don't remove friction too early — let the player feel it, then reward them for progressing past it.

See also: [[crate-manager-tier-progression]], [[storage-activation-gating-upgrade]]
