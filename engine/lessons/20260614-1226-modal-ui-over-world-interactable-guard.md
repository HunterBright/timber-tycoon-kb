---
title: Modal UI opened from a world-space interactable must guard the interaction handler
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-14'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fpp
- interaction
- modal-ui
- raycast
- input
- ui-event-system
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Modal UI opened from a world-space interactable must guard the interaction handler

## Severity
Medium - silent gameplay corruption (consumes resources / launches a second flow under the open UI). Easy to miss in code review, caught fast by an adversarial review pass.

## Context
FPP games commonly open a modal UI (transfer window, picker, shop) by pressing E / clicking a world object that is found via a center-screen raycast (`currentLookingAt`). The same projects usually **freeze the player camera + movement** while any modal/minigame is open (a single `IsAnyMinigameActive()`-style gate in PlayerController).

## The trap
Freezing the camera does **not** clear the raycast target. While the modal is open:
- `currentLookingAt` stays pinned to the very object you opened the modal on (camera is frozen pointing at it).
- The interaction handler (`PlayerInteraction.Update`) usually has early-return guards for *minigames*, but a brand-new modal UI is easy to forget to add to that list.
- Result: **LMB anywhere on screen** (including mis-clicks beside the panel) still routes to the frozen look-target - e.g. it ran the old "chop" path → consumed a log + launched the chopping minigame **on top of** the open picker. A **second E** re-ran `Show()` and silently reset the in-progress selection.

This is invisible for modals opened on a *different* object type than the action verb (e.g. a rack window: you're looking at a rack, so `TryAxeActionOnBlock()` returns false → benign). It only bites when the modal is opened on the **same object the look-action also targets** (a chopping block that is both the E-picker source AND the LMB-chop target).

## Fix
Add the new modal to the interaction handler's early-return guard list, mirroring the minigame guards:

```csharp
// PlayerInteraction.Update(), before CheckForInteractable()
if (MyModalUI.IsAnyOpen)   // non-creating static; do NOT use a lazy auto-creating Instance here
{
    HideAllPrompts();
    return;
}
```

Two gates are needed and they are different:
1. **PlayerController** movement/camera gate (`IsAnyMinigameActive`) - freezes look/move.
2. **PlayerInteraction** input gate - blocks E/LMB from firing on the frozen target.
A modal needs to be added to **both**; freezing the camera alone is not enough.

## Related patterns (also validated this session)
- **Reuse a single-action slot component for multi-button input without modifying it**: the project's `IconSlot` fires its `onClick` for *any* mouse button (ignores `eventData.button`). To get LMB=add / RMB=remove without touching the shared component, attach a second `IPointerClickHandler` (a tiny router) on the **same GameObject** and read `eventData.button`. Unity's `ExecuteEvents` invokes **every** `IPointerClickHandler` on the target GameObject, so both fire - leave the original's handler null (no-op) and let the router split by button.
- **ESC for modals: single handler only.** Don't let each modal read `Input.GetKeyDown(Escape)` in its own `Update` while a central PauseMenu also does - same-frame double-handling is execution-order-dependent (race). Route ESC for all modals exclusively through the PauseMenu's guard chain (close the modal + `return`), and give the modal a non-creating `IsAnyOpen` static for that check.
- **Lazy auto-creating `Instance` getter footgun**: if `Instance` auto-creates on access, never use it in read-only checks (ESC guard, movement gate) - every check would spawn the object. Expose a separate `public static bool IsAnyOpen => instance != null && instance.isOpen;` that reads the backing field without creating.

## Why transferable
Applies to any Unity FPP/3D game with raycast-based world interaction + modal/overlay UIs (tycoon, survival, immersive sim, shop-keeper). The "frozen camera leaves the look-target live" interaction is engine-level, not specific to this project's mechanics.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260707-1315-unity-interaction-raycast-blocked-by-noninteractable-collider|Single masked Physics.Raycast for "look-at to interact" gets eaten by a non-interactable collider on a masked layer]] - wspolne: fpp, interaction, raycast
- [[20260610-1345-unity-instance-setup-divergence|"Works for product A, dead for product B" = per-instance setup divergence, not code]] - wspolne: interaction, raycast
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] - wspolne: interaction, raycast
- [[20260724-1817-diegetic-button-overlap-steal|Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik]] - wspolne: input, raycast
- [[diegetic-3d-button-raycast|Diegetic 3D Button Raycast Pattern]] - wspolne: interaction, raycast
<!-- /POWIAZANE:auto -->
