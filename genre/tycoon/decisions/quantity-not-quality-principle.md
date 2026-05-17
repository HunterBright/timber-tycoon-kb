---
name: quantity-not-quality-design-principle
description: TT minigames affect output QUANTITY not QUALITY — species/machine tier determines quality, player skill determines how many items per cycle
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, tycoon, minigame, quality, quantity, progression]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Quantity-Not-Quality Design Principle

## Decision
Minigames in Timber Tycoon affect **output quantity**, not quality. Quality is determined by inputs (tree species + machine tier). Minigame execution (player skill) determines how many outputs per cycle.

## Context
TT has 5 minigames (tree cutting, sawmill, chipper, plank maker, pelletizer). The fundamental question: what does player skill change? Two options were considered.

## Options Considered

**Option A — Skill affects quality (REJECTED)**
- Bad execution → Regular Plank, Good → Fine Plank, Perfect → Premium Plank
- Problem: casual players can never get premium products → frustrating, feels punishing
- Problem: "quality" implies different visual models per grade → 3× asset complexity
- Problem: balance nightmare (Premium Plank worth 3×? What about Spruce Premium vs Oak Regular?)

**Option B — Skill affects quantity (SELECTED)**
- Bad execution → 1 Plank, Good → 2 Planks, Perfect → 3 Planks (same species/tier)
- Quality determined solely by inputs: Oak Plank always worth more than Spruce Plank regardless of skill
- Casual player: fewer planks per cycle (1), but never "bad" planks
- Skilled player: 3× output → time efficiency advantage

## Design Implications

**Species = quality tier:**
- Spruce (common), Birch (valuable), Oak (premium), Maple (similar to Oak)
- Player earns more by diversifying tree types, not by getting better at minigames

**Machine tier = quality cap:**
- Tier 1 PlankMaker → only Basic Plank. Tier 2 → Fine Plank. Player must upgrade.
- Minigame performance can't unlock what tier doesn't allow.

**NPC workers always produce "Good" (2x output):** consistent performance without skill variance. Late-game upgrade unlocks "Perfect" (3x).

## Why This Works
Separates "skill progression" from "content progression." Casual players feel successful (same product, fewer per cycle). Skilled players feel rewarded (time efficiency, same item). No "bad" items to demotivate.

## See also
[[carry-capacity-progression-sprint]], [[worker-output-quality-distribution]]
