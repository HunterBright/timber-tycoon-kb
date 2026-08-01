---
title: Diegetic 3D Button Raycast Pattern
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ui
- raycast
- minigame
- interaction
- diegetic
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Diegetic 3D Button Raycast Pattern

## When to use
Physical buttons in 3D world space (machine panels, kiosks, arcade controls) that the player interacts with via E key or mouse click. When you want "press a real button" instead of "click a UI overlay."

## Steps
1. Each button: separate GameObject with MeshCollider, named `Button_Green`/`Button_Red`/`Button_Yellow`
2. Root machine: BoxCollider for E-interaction (standard `IInteractable`)
3. **Critical**: DISABLE root BoxCollider during minigame - otherwise raycast hits root first and never reaches button colliders
4. Player camera raycast: `Physics.Raycast(camera.position, camera.forward, hit, range, interactableLayer)`
5. Check `hit.collider.parent` for `IInteractable` or button-specific component
6. After minigame end: RE-ENABLE root BoxCollider (restores E-interaction)

Pulse glow on buttons: URP/Lit `_EmissionColor` via MaterialPropertyBlock (per-instance, not shared material).

Ensure buttons are not on `IgnoreRaycast` layer (layer 2) - `Physics.DefaultRaycastLayers` skips it.

## Why this works
Diegetic UI (physical buttons) is more immersive than canvas overlays. Root collider must be disabled during minigame to avoid swallowing the raycast before it reaches the smaller button colliders.

## Trade-offs
More setup per button (individual MeshCollider, layer assignment). Worth it for the UX: player feels like they're pressing a physical button.

## Variants
Same pattern for: kiosk interactable ([[kiosk-interactable-cube-placeholder]]), machine on/off switches, lockbox combinations.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260713-1430-probe-visibility-by-rotating-rays-not-the-object|Sonda widoczności: obracaj PROMIENIE, nie obiekt]] - wspolne: minigame, raycast
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] - wspolne: minigame, raycast
- [[20260610-1345-unity-instance-setup-divergence|"Works for product A, dead for product B" = per-instance setup divergence, not code]] - wspolne: interaction, raycast
- [[20260614-1226-modal-ui-over-world-interactable-guard|Modal UI opened from a world-space interactable must guard the interaction handler]] - wspolne: interaction, raycast
- [[20260707-1315-unity-interaction-raycast-blocked-by-noninteractable-collider|Single masked Physics.Raycast for "look-at to interact" gets eaten by a non-interactable collider on a masked layer]] - wspolne: interaction, raycast
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] - wspolne: interaction, raycast
<!-- /POWIAZANE:auto -->
