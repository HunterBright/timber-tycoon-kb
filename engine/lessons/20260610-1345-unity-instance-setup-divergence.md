---
title: '"Works for product A, dead for product B" = per-instance setup divergence, not code'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- interaction
- raycast
- scene-setup
- data-driven
- debugging
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# "Works for product A, dead for product B" = per-instance setup divergence, not code

## Symptom
A shared interaction system works for one content instance (firewood rack: prompt OK)
and is silently dead for another of the same kind (chip-bag rack: no prompt at all).
Second occurrence in one project (before: stump prefab missing collider while log
prefab had one - digging impossible).

## Root cause pattern
Shared code is universal (one `StorageRackInteractable` serves all families), but each
scene instance must be individually "commissioned": active GameObject + collider the
raycast can hit + the interactable component + conventional layer. The broken instance
was a logical container only (data component wired by direct reference), never armed
for interaction - and every missing piece fails SILENTLY (raycast just passes through).

## Diagnostic recipe (validated, fast)
1. Don't read the shared code first - DIFF the broken instance against the working twin:
   dump for both: activeSelf/activeInHierarchy, layer, full component list, every
   collider in hierarchy (type/enabled/isTrigger/layer/bounds), wiring references.
   An editor script dumping all instances side-by-side found it in one pass.
2. Confirm at runtime with a synthetic raycast replicating the player's exact
   parameters (origin/direction/mask/range) - report what it hits and what
   component chain it resolves. Shows precisely WHERE the chain breaks
   (here: ray flew through the disabled rack and hit the building wall behind it).

## Second lesson: data-driven unlock IDs with no runtime consumer
The disabled rack was "gated" by an unlock entry (`spawn_chipbag_02`) in a progression
asset - but NO code consumes `spawn_*` unlocks. Data schema implied a feature that was
never implemented, so the object waits forever for activation. When adding data-driven
IDs (unlock types, event keys), grep for a consumer; an ID with zero readers is dead
content that LOOKS designed. Corollary: registration-in-OnEnable means an inactive
object is also invisible to every registry-based system (storage UI, managers) - one
disabled flag breaks multiple systems at different layers.

## Prevention
- When commissioning content N of a kind, copy the full component checklist from the
  working instance (or use a setup editor script), not just the data component.
- A "works for one, dead for the other" bug report should trigger instance-diff first.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260614-1226-modal-ui-over-world-interactable-guard|Modal UI opened from a world-space interactable must guard the interaction handler]] - wspolne: interaction, raycast
- [[20260707-1315-unity-interaction-raycast-blocked-by-noninteractable-collider|Single masked Physics.Raycast for "look-at to interact" gets eaten by a non-interactable collider on a masked layer]] - wspolne: interaction, raycast
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] - wspolne: interaction, raycast
- [[diegetic-3d-button-raycast|Diegetic 3D Button Raycast Pattern]] - wspolne: interaction, raycast
<!-- /POWIAZANE:auto -->
