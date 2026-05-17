---
name: building-progression-instant-spawn
description: Building expansions appear INSTANTLY post-purchase (1.5s fade-in only). Realistic construction time rejected — tycoons reward decisions immediately. "I bought it, it's there."
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, building, progression, instant, tavern-manager]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Building Progression — Instant Spawn Post-Purchase

## Decision

**Chosen:** Building expansions appear instantly after purchase. Visually: 1.5s material fade-in + dust VFX + construction sound. Functionally: wing is immediately accessible.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| Realistic construction | NPCs build over 5-10 in-game minutes (Banished / Frostpunk style) | REJECTED |
| Instant spawn with fade-in | Pre-placed wing fades in on purchase (#084) | SELECTED |

## Why instant wins

**Realistic construction fails for tycoons:**
- Interrupts the core gameplay loop — player must wait instead of play
- Players save-scum to skip construction anyway (reveals they don't want to wait)
- "Realism" is not TT's aesthetic (low-poly, stylized, arcade-ish)
- Decision→reward gap should be short in tycoons (Tavern Manager, Supermarket Sim, Farmer's Dynasty all instant)

**Instant spawn with cosmetic fade-in gives best of both worlds:**
- Immediate gameplay reward (new workstations accessible now)
- Visual "something happened" feeling via 1.5s fade
- Construction sound + dust VFX sell "construction" without time cost
- Inspired by: Tavern Manager (instant building reveal), Schedule I (immediate property upgrades)

## Sawmill wing order

1. **Strefa 1** — visible at game start (default)
2. **Strefa 2** — right wing, unlocked mid-game (~level 5 upgrade)
3. **Strefa 3** — left wing, unlocked late-game (~level 10 upgrade)

## Consequences

Wing prefabs are pre-placed in scene at level design time, inactive until purchased. Zero runtime positioning logic. See [[wing-snap-points-modular-fade-in]] for technical implementation.

See also: [[wing-snap-points-modular-fade-in]], [[player-built-vs-purchased-dichotomy]]
