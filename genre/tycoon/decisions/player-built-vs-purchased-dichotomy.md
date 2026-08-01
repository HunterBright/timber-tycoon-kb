---
title: Player-Built vs. Purchased Dichotomy
type: decision
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- game-design
- building
- player-agency
- scope
applies_to:
- unity-projects
source: ''
description: Only ONE thing is "built" by player (Counter_Broken → Fixed repair). Everything else (wings, machines, kiosks) is purchased and instantly appears. Limits scope, focuses meaning.
severity: high
suggested-category: genre/tycoon/decisions
name: player-built-vs-purchased-dichotomy
---

# Player-Built vs. Purchased Dichotomy

## Decision

**Chosen:** Exactly ONE object in the game is "built" by the player. Everything else is purchased and appears instantly.

## Built by player (manual, tutorial moment)

**Counter_Broken → Counter_Fixed:**
1. Player clears debris (6 piles, single-click each) → planks drop
2. Player picks up 1 plank
3. Player holds E at Counter_Broken for ~2s → Counter_Fixed appears

That's it. The ONLY moment where the player "builds something."

## Purchased + instant appearance (everything else)

- Sawmill wings (Strefa 2, Strefa 3)
- Machines (Chipper, PlankMaker, Pelletizer, future)
- Kiosks and interaction points
- Workers
- Vehicle upgrades
- Storage rack upgrades

All follow [[building-progression-instant-spawn]] pattern.

## Why limit player-built to one

**Scope argument:** Each player-built mechanic requires:
- Custom interaction flow (hold E / minigame / multi-step)
- Custom animation or VFX
- Tutorial explanation
- Testing per edge case

One = manageable. Five = a crafting game (not what TT is).

**Design argument:** The tutorial repair moment is powerful precisely because it's unique. If every upgrade required manual labor, the tycoon feel disappears - the game becomes work.

## Future projects

Games where "building things manually" IS the core mechanic (e.g., Eskimo Simulator: build igloo, craft sleds, construct dog pens) can scale player-built. In those games, the manual build IS the loop, not an edge case.

In tycoons: purchase = agency. Manual build = one special moment per game.

See also: [[building-progression-instant-spawn]], [[debris-cleanup-single-click-drop]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[building-progression-instant-spawn|Building Progression - Instant Spawn Post-Purchase]] - wspolne: building, game-design
- [[ui-phase-4-catalog|UI Phase 4 Catalog (12 Systems)]] - wspolne: scope, game-design
- [[vfx-wycofane-decision|VFX Wycofane - Sawdust, Kurz, Liście Cut from MVP]] - wspolne: scope, game-design
<!-- /POWIAZANE:auto -->
