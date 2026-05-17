---
name: tool-tier-replacement-decision
description: Tool upgrade (Axe T1→T2→T3) permanently REPLACES previous tier. Player never carries multiple tool tiers. Eliminates inventory bloat and "which axe?" decision fatigue.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, tool, tier, upgrade, inventory]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Tool Tier Replacement — NOT Multiple Tiers in Inventory

## Decision

**Chosen:** Tool upgrade PERMANENTLY replaces the old tier. T2 replaces T1. T3 replaces T2. Old tier is gone.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| Keep multiple tiers | T1, T2, T3 coexist in toolwheel (Stardew Valley approach) | REJECTED |
| Tier replacement | New tier replaces old. Old disappears. | SELECTED |

## Why replacement wins

Keeping multiple tiers introduces:
1. **Inventory bloat**: toolwheel grows with each upgrade
2. **Decision fatigue**: "which axe for this tree?" — player overthinks, loses flow
3. **Unclear upgrade value**: T1 still works after buying T2 — was the upgrade worth it?
4. **Save complexity**: must track all tier ownership per tool type

Replacement gives:
- Clear progression arc: "I have a better axe now"
- Upgrade feels **irrevocable** — commitment signals value
- Tycoon philosophy: always moving forward, no backsliding

## Implementation notes

```csharp
// On tier purchase:
player.toolManager.UpgradeAxe(newTier);
// Internally: destroy old tool GameObject, spawn new tier model
// SaveManager stores only currentTier int, not tier history
```

- Toolwheel updates to show new tier model immediately
- Viewmodel (hand-held model) swaps to new tier
- A T3 player cannot downgrade to T1 (irrevocable by design)
- Save/load: only `int currentAxeTier` persisted

## Consequences

Applies to: Axe, Shovel, future tools. Any tool system in TT follows this pattern. When designing a new tool: it starts at T1, player upgrades to T2, T1 is permanently replaced.

Future projects: same pattern recommended unless inventory management IS the game mechanic (not relevant for tycoons).

See also: [[tool-wheel-ux-pattern]], [[quantity-not-quality-principle]]
