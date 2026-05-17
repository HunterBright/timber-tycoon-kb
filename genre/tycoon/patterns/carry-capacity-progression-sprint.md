---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [game-design, tycoon, progression, carry, sprint, npc]
date: 2026-05-17
status: draft
---

# Carry Capacity Progression + Sprint Advantage

## When to use
Tycoon games where the player physically carries items between stations. Player must feel meaningfully faster than automated alternatives (NPC workers), or they have no incentive to participate in the loop.

## Steps

**Carry capacity upgrade tiers:**

| Level | Polish name | Items | Visual |
|-------|-------------|-------|--------|
| 0 (start) | — | 1 log | Single log in arms |
| 1 | Rękawice robocze | 3 logs | Bundle under arm |
| 2 | Szelki drwala | 5 logs | Stack on shoulder |
| 3 | Siła weterana | 8 logs | Large stack (endgame) |

**NPC worker baseline:** always 3 logs, walk speed only (no sprint), +200ms delay at storage rack ("searching" animation).

**Player sprint:**
```csharp
float speed = Input.GetKey(KeyCode.LeftShift) ? runSpeed : walkSpeed;
// walkSpeed: 5 m/s, runSpeed: 10 m/s (2× multiplier)
// Sprint available regardless of carry amount (no speed penalty for carrying)
```

**Sprint cost:** Stamina system considered and rejected. Sprint is free — tycoon games punish AFK, not active play.

**Player vs NPC comparison at Level 3 + sprint:**
- Player: 8 logs, 10 m/s, 0 delay = 80 log-meters/second effective throughput
- NPC: 3 logs, 3 m/s, 0.2s delay = ~9 log-meters/second effective throughput
- Player is ~8.8× faster than NPC at max progression

**Late-game balance:** player isn't expected to carry ALL logs themselves. Hybrid play: player runs high-value premium routes, NPCs handle bulk Spruce volume.

## Why this works
Sprint + capacity creates a clear player advantage throughout the game. Even at Level 0 (1 log), sprint means ~3× faster than walking NPC with 3 logs. Player never feels "I should just AFK."

## Trade-offs
- No stamina system: player can sprint indefinitely. Keep it. Fatigue in tycoon = frustrating, not engaging.
- Level 0 = 1 log: early game humbling moment. Intentional — teaches upgrade system value. Keep first upgrade early (within 5 minutes).
- NPC `StorageDelay`: 200ms is subtle enough to feel "realistic" (NPC fumbles with rack), large enough to be measurable at scale.

## Variants
- **Species-specific carry limits:** heavy logs weigh more. Add variety, adds complexity. Rejected for TT (cognitive load too high)
- **Vehicle carry**: truck carries 24-60 logs (see TruckData SO). Separate system — player drive the truck, not carry the logs.

See also: [[quantity-not-quality-design-principle]], [[order-fulfiller-interface]]
