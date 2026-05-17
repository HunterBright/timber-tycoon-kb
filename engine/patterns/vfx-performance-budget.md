---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, vfx, particle-system, performance, budget]
date: 2026-05-17
status: draft
---

# VFX Performance Budget

## When to use
Any scene with more than 2-3 active particle systems. Without a budget, "subtle dust effect" on 50 simultaneous events → 5000 particles → 30fps. With budget, 60fps locked.

## Steps

**Timber Tycoon budget:**
- **500 particles total** across all active systems at any given frame
- **10 active ParticleSystems** simultaneously

**VFXManager singleton enforcement:**
```csharp
// Before spawning any VFX
if (!VFXManager.Instance.RequestSpawn(particleCount)) return; // denied
// spawn particle system, deduct from budget
// on stop: VFXManager.Instance.Release(particleCount)
```

**Per-system configuration:**
- Set `Max Particles` in ParticleSystem Inspector to a safe per-category value
- Calculate: `emissionRate × lifetime × maxConcurrentSystems ≤ categoryQuota`

**Naming convention:** `vfx_kategoria_opis` in `Assets/VFX/`
- `vfx_machine_sparks`, `vfx_machine_smoke`, `vfx_coin_sale`, `vfx_vehicle_exhaust`, `vfx_water_splash`

**Budget categories for Timber Tycoon:**
| Category | Max systems | Max particles each | Total |
|----------|-------------|-------------------|-------|
| Machine (sparks/smoke) | 4 | 50 | 200 |
| Sale coins | 2 | 30 | 60 |
| Vehicle exhaust | 2 | 20 | 40 |
| Water splash | 1 | 100 | 100 |
| Reserve | — | — | 100 |

**VFXManager throttle policy:** if over budget, skip spawn (not queue — queuing causes delayed particle flood). Critical VFX (sale coins) get priority; ambient VFX (smoke) get skipped first.

## Why this works
Setting a hard scene-wide cap prevents the worst-case scenario: dozens of tree cuts in rapid succession each spawning an uncapped particle system. Enforcing at the manager level means individual VFX prefabs don't need to be aware of each other.

## Trade-offs
- Budget is approximate (Max Particles ≠ actual particles at a given frame, but close enough)
- VFX may skip on high-action moments — design VFX as "nice to have" enhancements, not gameplay feedback
- Budget numbers are project-specific; these are TT values, not universals

## Variants
- **No budget manager** (small projects): just set low Max Particles on each prefab and accept occasional skips
- **Priority queue** instead of skip: queue low-priority VFX and play them when budget frees up (adds complexity)

See also: [[vfx-trigger-pattern]], [[game-event-so-event-channel]]
