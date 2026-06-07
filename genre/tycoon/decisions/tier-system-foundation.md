---
name: tier-system-foundation
description: SOURCE OF TRUTH for Timber Tycoon tiers. Trees have 3 tiers (T1/T2/T3, 10 species). Most products are NOT tier-graded (1 species = 1 SKU, no Basic/Fine/Premium planks). Firewood is the ONLY graded product and its tier comes from the TREE tier. Machine tier = a GATE on tree tiers, NOT a quality cap. Corrects errors in quantity-not-quality-principle.md and customer-tier-system.md.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, tiers, economy, trees, machines, products, source-of-truth]
  severity: high
  date: 2026-06-07
  status: draft
  applies_to: [unity-projects]
---

# Tier System Foundation — Trees, Products, Machines (SOURCE OF TRUTH)

This entry governs how tiers, products, and machines relate in Timber Tycoon. Where any other KB entry conflicts with this, THIS wins. It corrects errors in `quantity-not-quality-principle.md` and `customer-tier-system.md` (see §4).

## 1. Tree tiers
Tier is a property of the TREE, not the machine. Three tiers, ten species:

| Tier | Species |
|------|---------|
| T1 | Spruce, Birch, Oak, Maple |
| T2 | Acacia, Beech, Poplar |
| T3 | Mahogany, Teak, Ebony |

Demo ships T1 only. T2/T3 are post-demo content adds (new SO + models, zero code).

## 2. Products (SKU taxonomy)

| Product | SKU count | Tier-aware? | Logic |
|---------|-----------|-------------|-------|
| Logs | 10 (per species) | yes | NOT for sale — always processed. Log's tier GATES which machine accepts it. |
| Planks | 10 (per species) | NO | 1 species = 1 SKU. No tier variants. Species sets price (Oak > Spruce), NOT a quality grade. |
| Firewood | 3 (per TREE tier) | yes (from tree) | T1 → Basic, T2 → Good, T3 → Premium. Species dissolves into the tier. |
| Wood chips | 1 | no | Species-agnostic, sellable, intermediate. |
| Pellet | 1 | no | Made from wood chips. Always same quality. |
| Mulch | 1 | no | Species-agnostic. Made from BARK. Dual-use: sellable AND consumed to fertilize trees → shorter growth time (see §5). |
| Bark | 1 | no | Species-agnostic byproduct (of PlankMaker). Sell raw, or convert to Mulch. |
| Furniture | ~6–8 recipes | — | Out of scope (post-demo). Mix of plank species → furniture. |
| Stump | — | — | NOT for sale — cleanup only (frees the planting spot). |

## 3. Machine logic — the foundation
Machine tier = a GATE on tree tiers. It does NOT change output quality.
- Machine tier determines which tree tiers it can process; cannot process wood above its own tier.
- Machine tier does NOT improve the product (no "Fine Plank"). It only widens the range of trees accepted (and likely throughput/speed — not yet locked).
- Example: PlankMaker T1 → T1 trees only. T2 → T1+T2. T3 → all.

## 4. Corrections to existing KB
WRONG claims elsewhere, to be changed to match this entry:
- `quantity-not-quality-principle.md`: "Machine tier = quality cap (T1 → Basic Plank, T2 → Fine Plank)" — FALSE. Machine tier is a tree-tier gate, not a quality upgrade. No Fine Plank exists.
- `quantity-not-quality-principle.md`: "Species = quality tier" — misleading. Species = separate SKU at a different price (Plank_Oak > Plank_Spruce). No Basic/Fine/Premium gradation.
- `customer-tier-system.md`: VIP example "20 Premium Pellets" — no Premium Pellet exists. Pellet is a single SKU.

Still CORRECT in `quantity-not-quality-principle.md`: minigame skill affects QUANTITY (1/2/3 per cycle), not quality; worker output 2.1 avg. The ONLY tier-graded product is Firewood, and its tier comes from the TREE tier — not skill, not species.

## 5. Mulch / fertilizer loop (system to design)
Mulch is made from Bark. It is BOTH sellable AND a consumable: applying it to a tree shortens that tree's growth time. Make-or-sell tension (sell for cash vs. spend to accelerate supply). Exact mechanic (reduction per bag, consumed 1:1?, application UI, per-tier behaviour) NOT yet specified — separate design task. Out of demo scope; captured so it isn't lost. NOTE: code currently names this product "Fertilizer" (ProductType.Fertilizer / FertilizerMaker) — rename to Mulch pending.

## 6. Demo scope vs. future
- Demo: T1 trees only; Firewood Basic only; planks per T1 species; wood chips, pellet, mulch, bark. Machine-tier gating exists but only T1 reachable.
- Post-demo: T2/T3 trees, Firewood Good/Premium, machine T2/T3 gates, mulch→growth-accel, Furniture.

## See also
- [[quantity-not-quality-principle]] (corrected by this entry)
- [[customer-tier-system]] (corrected by this entry)
- [[storage-rack-family-system]] (Plank rack maxDistinctStacks must cover species count)
- [[reputation-levels-data-driven]]
