---
title: Don't build a full-screen dim/scrim by reusing your skinnable panel factory
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- ui
- 9-slice
- sprite
- skinning
- modal
- overlay
- alpha
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Don't build a full-screen dim/scrim by reusing your skinnable panel factory

## The trap
You have a code-built UI with a central factory method (e.g. `UIFactory.CreatePanel`)
that, once a 9-slice `panelSprite` is assigned in the skin, renders `Image.type = Sliced`
with that sprite. The full-screen modal *dim* (the dark scrim behind a window) was also
built with `CreatePanel`, just passing a black tint like `(0,0,0,0.65)`. It looks fine
while every skin sprite slot is still null (flat-color fallback), so the dim ships as a
plain dark rectangle. The trap: it keeps working *only until* the first real panel sprite
is dropped in.

## Why it fails
`Image` tint multiplies the sprite's RGB **and** alpha; it does not replace shape.
- The black tint zeroes the *color* (so people assume "tint makes it black, no-op").
- But the scrim's **alpha and silhouette now follow the sprite's alpha channel**. A wooden
  / bark / rounded panel sprite has a transparent background and is 9-sliced, so the
  full-screen overlay stops being a uniform rectangle: transparent regions stop dimming,
  and the 9-slice border geometry gets stretched across the whole screen.
Net: assigning a panel sprite silently breaks every dim that shares the panel factory.

## Symptoms
- The screen-darkening behind a modal becomes patchy / shows panel edge artifacts after a
  skin sprite is assigned - even though "only a tint" is passed.
- It worked perfectly during the flat-color phase of a UI reskin, then regressed the moment
  the first 9-slice sprite went into the skin.

## Correct approach
Give the scrim its own factory method that is **skin-independent** and always solid:
a full-rect `Image` with `sprite = null`, `Image.type = Simple`, `color = color`, anchors
0,0-1,1. It is a scrim, not a panel - it must never read the skin's `panelSprite`.
In Timber Tycoon: `UIFactory.CreateDimOverlay(Transform parent, Color color)`; the rack
transfer window's dim call was swapped from `CreatePanel(...)` to it. Rule of thumb during
any flat-to-sprite UI reskin: audit every caller of the panel factory and split out the
ones that are actually scrims/dimmers *before* assigning the first sprite, because the
flat-color phase hides the bug.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260613-0625-9slice-ppu-must-scale-to-target-rect-not-stay-100|A large 9-slice sprite at PixelsPerUnit=100 breaks because its fixed corners exceed the panel]] - wspolne: sprite, 9-slice, ugui
- [[20260612-1845-blender-9slice-ui-sprites|Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)]] - wspolne: sprite, 9-slice, ui
- [[20260612-1330-getbuiltinresource-extra-null|Resources.GetBuiltinResource zwraca NULL dla sprite'ów UI (builtin-EXTRA) - pasek Filled rysuje się jako pełny quad]] - wspolne: sprite, ugui
- [[20260714-1250-ugui-okno-rosnace-pod-tresc|Lekcja: okno UI rosnace pod tresc - clamp do ekranu NIE zmniejsza tresci]] - wspolne: modal, ugui
- [[20260616-1532-unity-9slice-contentsizefitter-pollution|9-slice Image na obiekcie layoutu zaniża ContentSizeFitter (panel rośnie do sumy borderów)]] - wspolne: 9-slice, ugui
- [[20260607-2016-ugui-filled-image-needs-sprite|A UGUI `Image` with `type = Filled` but no sprite ignores `fillAmount`]] - wspolne: ugui, ui
<!-- /POWIAZANE:auto -->
