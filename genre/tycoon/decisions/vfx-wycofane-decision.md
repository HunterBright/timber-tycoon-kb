---
title: VFX Wycofane - Sawdust, Kurz, Liście Cut from MVP
type: decision
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- game-design
- vfx
- scope
- deferred
- decision-record
applies_to:
- unity-projects
source: ''
description: 'Sawdust, tree-fall dust, and seasonal leaves VFX cut from MVP. Scope cut with explicit rationale - retained: vehicle exhaust, sawmill smoke, splash, gold coin sale.'
severity: medium
suggested-category: genre/tycoon/decisions
name: vfx-wycofane-decision
---

# VFX Wycofane - Sawdust, Kurz, Liście Cut from MVP

## Decision

**Cut from MVP:**
- Sawdust cloud on tree chop
- Kurz (dust cloud) on tree fall
- Seasonal liście (leaves falling, autumn)

**Retained:**
- Vehicle exhaust (active road feel)
- Sawmill smoke (machine running indicator)
- Splash (water zone interaction)
- Gold coin animation on sale (positive feedback)

## Rationale

Each VFX requires: particle system design, GPU performance tuning, audio sync, testing across platforms. Cost: ~4-8h per effect.

**Why sawdust/kurz/liście cut:**
- Not core to "tycoon" experience (vs. immersive sim or survival)
- VFX budget (#075) is limited for MVP
- Player attention in tycoon is on numbers, upgrades, flow - not on particle effects
- Likelihood of player noticing absence: low. Likelihood of player noticing performance drop from adding: high.

**Why retained VFX kept:**
- Vehicle exhaust: proves the road and traffic system is alive
- Sawmill smoke: communicates "machines are working" at a glance
- Splash: water zone interaction needs visual confirmation
- Gold coin: direct positive reinforcement on sale moment

## Revisit condition

Post-demo launch: if community feedback includes "game feels lifeless" or "trees don't feel real when cut" - re-add sawdust/kurz as first candidates. Use VFX budget pattern (#075) to evaluate fit.

## Lesson for future projects

Itemize ALL planned VFX. Score each on: gameplay value / dev cost / player noticeability. Cut bottom 50% from MVP. Retained VFX should all have clear gameplay communication function, not just visual flair.

See also: [[vfx-performance-budget]], [[vfx-trigger-pattern]]
