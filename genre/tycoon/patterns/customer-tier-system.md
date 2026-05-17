---
name: customer-tier-system
description: Three NPC customer tiers (Regular/Contractor/VIP) unlock via Reputation — spawn distribution scales with progression, VIP is a milestone moment
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [game-design, tycoon, customer, tier, progression]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Customer Tier System (Regular / Contractor / VIP)

## When to use
Tycoon games where customer value should scale with player progression. Starting with VIPs is overwhelming; only ever getting small orders feels stagnant. Tier system = visible progression milestone.

## Steps

**Customer tier definitions:**

| Tier | Order Size | Species | Revenue | Visual |
|------|------------|---------|---------|--------|
| Regular | 5–10 items | Common (Spruce) | $50–100 | Economy car |
| Contractor | 15–30 items | Often premium | $300–500 | Work truck |
| VIP | 50+ items | Premium specific | $1000+ | Luxury sedan |

**Spawn distribution per Reputation level:**

| Reputation Level | Regular | Contractor | VIP |
|-----------------|---------|------------|-----|
| 1–2 | 100% | 0% | 0% |
| 3–5 | 70% | 30% | 0% |
| 6–8 | 50% | 40% | 10% |
| 9–12 | 30% | 50% | 20% |
| 13 (max) | 20% | 50% | 30% |

**Implementation:**
```csharp
CustomerTier GetTierForSpawn() {
    var rand = Random.value;
    var dist = reputationManager.GetTierDistribution(); // returns (regular%, contractor%, vip%)
    if (rand < dist.vip) return CustomerTier.VIP;
    if (rand < dist.vip + dist.contractor) return CustomerTier.Contractor;
    return CustomerTier.Regular;
}
```

**VIP milestone moment — "Demo Day 4 FINAL PUSH":**
- First VIP customer spawns: $1500 mansion order (50 Oak Planks + 20 Premium Pellets)
- Designed as climactic moment — player has earned it through all prior progression
- Announce via Notification Queue: "New customer type: VIP! Premium orders available."

**Visual differentiation:** different `CarVariants SO` per tier. Regular = economy car colors, VIP = dark/luxury palette. Player learns to read tier from car model before NPC even parks.

## Why this works
Regular customers early game = manageable orders, player learns systems. VIP unlocks = tangible reward for reputation building. The ramp from 0% VIP to 30% VIP at max rep = always something to work toward.

## Trade-offs
- Order fulfillment risk: VIP orders are large (50+ items). If inventory falls short, partial fulfillment or failure. Design safety valve: VIP orders should be achievable with 30-minute stockpile
- Species specificity: VIPs request specific species. If player hasn't diversified tree types, they can't fulfill VIP orders → soft skill gate. Intended.
- Time-limited VIP: "order expires in 3 in-game hours" creates pressure. Implemented in late-game only (not demo)

## Variants
- **Loyalty tiers:** returning customers get discounts/priority. Adds retention loop, higher complexity.
- **Seasonal specials:** VIP orders only during specific in-game events. Adds calendar strategy.

See also: [[pipeline-style-npc-spawn]], [[reputation-levels-data-driven]]
