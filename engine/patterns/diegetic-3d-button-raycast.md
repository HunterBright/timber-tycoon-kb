---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, ui, raycast, minigame, interaction, diegetic]
date: 2026-05-17
status: draft
---

# Diegetic 3D Button Raycast Pattern

## When to use
Physical buttons in 3D world space (machine panels, kiosks, arcade controls) that the player interacts with via E key or mouse click. When you want "press a real button" instead of "click a UI overlay."

## Steps
1. Each button: separate GameObject with MeshCollider, named `Button_Green`/`Button_Red`/`Button_Yellow`
2. Root machine: BoxCollider for E-interaction (standard `IInteractable`)
3. **Critical**: DISABLE root BoxCollider during minigame — otherwise raycast hits root first and never reaches button colliders
4. Player camera raycast: `Physics.Raycast(camera.position, camera.forward, hit, range, interactableLayer)`
5. Check `hit.collider.parent` for `IInteractable` or button-specific component
6. After minigame end: RE-ENABLE root BoxCollider (restores E-interaction)

Pulse glow on buttons: URP/Lit `_EmissionColor` via MaterialPropertyBlock (per-instance, not shared material).

Ensure buttons are not on `IgnoreRaycast` layer (layer 2) — `Physics.DefaultRaycastLayers` skips it.

## Why this works
Diegetic UI (physical buttons) is more immersive than canvas overlays. Root collider must be disabled during minigame to avoid swallowing the raycast before it reaches the smaller button colliders.

## Trade-offs
More setup per button (individual MeshCollider, layer assignment). Worth it for the UX: player feels like they're pressing a physical button.

## Variants
Same pattern for: kiosk interactable ([[kiosk-interactable-cube-placeholder]]), machine on/off switches, lockbox combinations.
