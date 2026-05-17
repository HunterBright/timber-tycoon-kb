---
name: rack-architecture-decision
description: Storage racks: 3 options analyzed — shared BagRack with capacity upgrades, dedicated rack per product, visual-only swap. Decision open; recommendation is Option B (dedicated racks).
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, storage, architecture, decision-record]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Rack Architecture Decision (3 Options)

## Decision

**Status:** Open — pending Creative Director (Hunter) decision.
**Recommendation:** Option B — Dedicated rack per product type.

## Options considered

### Option A — Shared BagRack with capacity upgrades
- One rack holds all bag types (pellet + fertilizer + furniture bags)
- Upgrades increase slot count
- **Pro:** Simple model, player clearly feels "more slots"
- **Con:** Visual chaos late game — a single rack mixing pellets, fertilizer, furniture = no visual clarity

### Option B — Dedicated rack per product type (RECOMMENDED)
- PelletRack, FertilizerRack, FurnitureRack are distinct GameObjects
- Each unlocked as purchased upgrade
- **Pro:** Visual clarity — sawmill looks like a showroom, each rack is a landmark
- **Con:** More racks = more floor space needed; late game sawmill may feel crowded

### Option C — Visual upgrade only (model swap)
- Same rack, just swaps to a fancier model when upgraded
- **Pro:** Zero gameplay change, easy to implement
- **Con:** Player buys upgrade but nothing changes functionally — "what did I just buy?"

## Recommendation rationale

Option B wins because Timber Tycoon's core appeal is the sawmill as a "place" — the player should walk around and see distinct areas for each product. A dedicated PelletRack next to the Pelletizer communicates workflow visually. Option A muddles this by mixing everything. Option C provides no upgrade satisfaction.

## Consequences

If B chosen: plan floor space in sawmill layout (Strefa 2 and 3 expansions should accommodate dedicated racks). Each rack type needs its own FBX model and StorageFamily enum entry.

See also: [[storage-rack-family-system]], [[global-router-storage-pattern]]
