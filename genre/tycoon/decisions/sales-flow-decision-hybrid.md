---
name: sales-flow-decision-hybrid-d
description: Sales flow decision — 4 options (player-only, auto-NPC, mixed, hybrid) analyzed, chose D hybrid. Player + NPC worker serve queue simultaneously. Foundational for all worker design.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, sales, customer-flow, decision-record]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Sales Flow Decision — Hybrid D (Player + NPC Side by Side)

## Decision

**Chosen:** Option D — Hybrid. Player AND NPC worker both serve the customer queue simultaneously.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| A | Player serves all customers | REJECTED — overwhelmed late game |
| B | NPC worker handles everything | REJECTED — player goes AFK |
| C | Player chooses per order: manual or auto | REJECTED — decision fatigue |
| D | Both active simultaneously | SELECTED |

## Why D wins

**Option A** fails at scale: late-game with 50+ customers/day = player can't keep up. Game becomes stressful, not fun.

**Option B** removes the core loop: if an NPC handles everything, the player is just watching a screensaver.

**Option C** introduces meta-decisions per order: "should I do this one myself or let the NPC?" — adds friction to the core loop.

**Option D** gives players ongoing agency while letting workers handle overflow:
- Player advantages: sprint speed, higher carry capacity (8 vs 3 items), no storage delay (see [[order-fulfiller-interface]])
- NPC advantages: 24/7 availability, frees player attention for other tasks
- Late game: hire 2nd NPC worker → 3× throughput without removing player engagement

## Implementation notes

```
Counter has 2 service slots (Counter_1, Counter_2)
Both player and NPC worker can serve different customers at the same time.
OrderQueue.AssignNextFulfiller() picks idle fulfiller (player priority if active)
```

Throughput scaling:
- 1 player alone: ~4 customers/min
- +1 NPC: +2 customers/min
- +2 NPC: +4 customers/min (player can shift to restocking)

## Consequences

All subsequent worker architecture assumes hybrid fulfillment. `IOrderFulfiller` interface designed for asymmetric implementations. Never regress to Option A or B without revisiting this ADR.

See also: [[order-fulfiller-interface]], [[customer-tier-system]], [[worker-simulate-work-cycle]]
