---
type: pattern
project: Timber Tycoon
suggested-category: design/patterns
tags: [economy, gating, content-pool, emergent-design, tycoon, npc-orders]
date: 2026-06-07
status: draft
---

# Gate a content pool by runtime availability, not explicit unlock flags

## Context
A shop/tycoon needs to control *what customers can order* as the player unlocks
production (e.g. no pellet orders until the player buys a pelletizer and makes pellets).
The naive approach is an explicit unlock system: per-product `isUnlocked` flags toggled
when a machine is purchased, plus the bookkeeping/save-migration that implies.

## Pattern
Keep ONE static, complete "menu" of every sellable product, but make the order
generator **filter it by current warehouse stock at order time**:

```
foreach (orderable in pool)
    if (storage.GetAmount(orderable.product, species) > 0)
        candidates.Add(orderable);   // stock 0 -> silently never ordered
if (candidates.Count == 0) return null;
amount = min(stock, maxPerItem);     // order never exceeds what's on hand
```

Now "unlocking" is **emergent**: an entry produces orders the moment the player
first produces that product, and stops when stock runs out. Listing `PelletBag` in
the pool is harmless until a pelletizer actually outputs pellets. No unlock flags,
no save migration, no machine->product wiring.

## Trade-offs
- (+) Zero unlock bookkeeping; "produce it = unlock it" is automatic and self-saving.
- (+) The seeded pool doubles as the design's price/reputation table in one place.
- (−) Can't model "produced but deliberately not yet orderable" — availability is
  strictly tied to stock. If you need a product gated behind something *other* than
  stock (story, reputation tier), you still need an explicit gate on top.
- (−) Designers must remember the pool is a *candidate menu*, not a guarantee — an
  entry with no matching production path is silently dead.

## Timber Tycoon instance
`NPCOrderGenerator.GenerateFromWarehouse` is warehouse-filtered; the order pool is
seeded once by an editor menu (`SetFirewoodOnlyOrderables` / "Set Demo Orderables")
with 8 producible entries (firewood, chips, bark, 4 plank species, pellet bag).
Pellet/plank orders simply don't appear until those machines run. Reputation value
lives per-entry in the same seeder.
