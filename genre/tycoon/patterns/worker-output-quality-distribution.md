---
name: worker-output-quality-distribution
description: Worker default output — 20% Bad/50% Good/30% Perfect random roll. perfectQuality flag = always Perfect. Late-game upgrade, never makes workers OP vs. skilled player.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [game-design, tycoon, worker, quality, distribution, progression]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Worker Output Quality Distribution

## When to use
Tycoon games where automated workers produce outputs that the player can also produce manually. The distribution must keep workers "useful but not dominant" — players should still want to participate.

## Steps

**Output quality roll:**
```csharp
public OutputQuality GetOutputQuality() {
    float roll = Random.value;
    if (roll < 0.20f) return OutputQuality.Bad;     // 20%: 1 output
    if (roll < 0.70f) return OutputQuality.Good;    // 50%: 2 outputs
    return OutputQuality.Perfect;                   // 30%: 3 outputs
}
```

**Expected output per cycle:** `0.20×1 + 0.50×2 + 0.30×3 = 0.2 + 1.0 + 0.9 = 2.1` (default worker)

**perfectQuality flag:**
```csharp
var quality = worker.perfectQuality ? OutputQuality.Perfect : worker.GetOutputQuality();
```
- Only achievable at Sawmill Level 13 upgrade ("Maestro Workforce")
- Worker at Perfect: always 3 outputs = 43% better than default (2.1 avg)

**Player comparison:**
- Unskilled player: ~50% Perfect rate → 2.5 avg output (better than default worker)
- Skilled player: ~60-70% Perfect rate → 2.6-2.8 avg (better than max worker)
- Max worker (3.0): matches skilled player when combined with higher `workSpeedMultiplier`

**Design intent:** "default worker is slightly worse than active player. Max worker equals active player. Both are needed."

**Tuneable:** distribution stored in `BalanceConfig.workerQualityDistribution` SO — designers can adjust without code changes.

## Why this works
20/50/30 is calibrated so workers feel useful (2.1 avg > 1 if player does nothing) but not dominant over engaged player (2.5+ for any active play). `perfectQuality` as endgame unlock gives meaningful upgrade goal at level 13.

## Trade-offs
- Random distribution: workers produce slightly different amounts each cycle. Acceptable variance for a tycoon (not slot machine). Add min-guarantee: "minimum 1 output per cycle" (already implied by Bad = 1)
- Balance sensitivity: shifting Bad to 0% and Perfect to 50% makes workers nearly as good as players early-game. Test carefully before any distribution change.
- Visual feedback: when worker produces "Perfect" output, show a small particle effect or "+3" floating text. Rewards players for watching automation.

## Variants
- **Flat average:** worker always produces 2 outputs (no RNG). Simpler, predictable but less interesting.
- **Skill-leveled workers:** each individual worker has a hidden "skill" float that improves over time (like Stardew NPC friendships) — adds progression, adds save complexity.

See also: [[worker-data-instance-split]], [[worker-simulate-work-cycle]], [[quantity-not-quality-design-principle]]
