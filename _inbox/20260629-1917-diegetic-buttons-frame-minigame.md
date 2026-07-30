---
title: Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-29'
project: Kerf - Sawmill Tycoon
tags:
- tycoon
- minigame
- diegetic-ui
- machine
- raycast
- camera-lock
- design-reconciliation
applies_to: []
source: ''
suggested-category: genre/tycoon/patterns
---

# Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)

## Problem it solves
A "functional buttons on the machine" request seems to conflict with a "skill minigame"
(mouse/timing) design — they look mutually exclusive. They're not, if you separate the
two roles: the buttons are the SESSION FLOW (operator console), the minigame is the SKILL.

## The pattern
A processing machine runs one session as a phase state machine driven by 3 diegetic 3D
buttons + one embedded skill minigame between them:

```
WaitForGreen → (green) arm / open intake
WaitForRed   → (red)   consume input + close + start the machine + ENTER minigame
[minigame]   → the actual skill challenge → verdict (Bad/Good/Perfect)
WaitForYellow→ (yellow) commit verdict → output to storage
```

Buttons = flow control; the minigame in the middle is whatever skill mechanic fits
(timing, balance, drag-tempo). Swapping the middle mechanic doesn't touch the button
shell. This is the SAME shape across multiple machines — clone the shell, vary the middle.

## Implementation notes (Unity)
- **Diegetic buttons:** named child meshes (`Button_Green/Red/Yellow`). Click via
  `Physics.RaycastAll` + filter by name, scoped to THIS machine's hierarchy
  (`transform.IsChildOf(machineRoot)`) so identically-named buttons on other machines
  in frame don't get hit. No need to disable the body collider if you filter by name.
- **Buttons baked into a shared atlas** (one material, colors in the texture) still
  light up: give each button renderer its OWN material instance at runtime
  (`renderer.material` + `EnableKeyword("_EMISSION")`) and pulse `_EmissionColor` per phase.
  You CAN detect a press on a cosmetic baked button; you just need the per-renderer
  instance to make it glow without touching the shared asset.
- **Colliders may be missing** if the FBX imported with `addColliders:0` — add a
  BoxCollider at runtime from the mesh bounds (×~1.6 for small targets) rather than
  baking colliders into the prefab.
- **Buttons gate input per phase** so they never fight the minigame: only read button
  clicks in the Wait* phases; during the skill minigame the mouse drives the mechanic.
- Reuse the camera-lock (save world pose → lerp → restore) and a vertical/horizontal
  shared gauge; don't rebuild per machine.

## Design-process lesson
Before treating two requests as a conflict, check whether an existing machine already
does both — here the conflict was illusory (the Pelletizer already framed its minigame
with green/red/yellow). Recon dissolved a "decision" that wasn't one.

## Evidence
Timber Tycoon: `PelletizerMinigameUI` (origin), `FertilizerMakerMinigameUI` (Phase 4,
cloned). Validated in Play Mode by Hunter.
