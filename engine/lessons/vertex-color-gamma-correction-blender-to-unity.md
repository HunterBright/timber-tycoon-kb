---
title: Vertex Color Gamma Correction Blender → Unity
type: lesson
status: needs-reproduction
confidence: medium
verified: '2026-07-30'
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- blender
- unity
- vertex-color
- gamma
- srgb
- linear
applies_to:
- unity-projects
- blender-pipelines
source: zweryfikowane reprodukcja i dokumentacja 2026-07-30, patrz AUDYT-SPORNYCH-WPISOW
severity: high
suggested-category: engine/lessons
time_lost: ''
audit_verdict: DO SPRAWDZENIA
---

# Vertex Color Gamma Correction Blender → Unity

> [!warning] Ten wpis zostal zweryfikowany 2026-07-30 i werdykt brzmi: **DO SPRAWDZENIA**
>
> Eksporter Blendera pozwala wybrac kolory jako sRGB albo liniowe, wiec reczne przeliczanie NIE jest uniwersalnie wymagane.
> U nas wzorzec przeliczania jest w trzech produkcyjnych skryptach, wiec dla naszej sciezki dziala.
>
> Pelne uzasadnienie, dowody i proponowane poprawki: [[AUDYT-SPORNYCH-WPISOW]].
> Tresc ponizej NIE zostala jeszcze przepisana - czytaj ja z ta uwaga.


## Problem
Vertex colors assigned in Blender appear noticeably darker in Unity. Bright grass green in Blender becomes muddy olive in Unity. Every terrain regeneration requires multiple iterations of "darker, no darker, no lighter" until you discover the gamma issue.

## Root cause
Blender stores vertex colors in linear space (0-1 floats, no gamma). Unity shaders (URP Lit, Custom/VertexColorLit) expect sRGB. The gamma mismatch causes all vertex colors to read darker than designed.

## Solution
Apply gamma correction in the Blender Python export script before writing vertex colors:

```python
def linear_to_srgb(c):
    if c <= 0.0031308: return c * 12.92
    return 1.055 * (c ** (1/2.4)) - 0.055
```

Apply per-channel before writing to the vertex color attribute.

Simpler approximation (close enough for low-poly): `c ** (1/2.2)`

Verify in Unity Play Mode Game view - Editor Scene view can lie.

## What didn't work
Assigning vertex colors directly from linear Blender values - all colors darker than designed.

## Transferability
Any pipeline that carries vertex colors from Blender to Unity (or any sRGB-space engine) will hit this. Bake the conversion into the export script once; never manually compensate colors again.

## Related
- [Desaturated colors for low-poly](desaturated-colors-for-low-poly.md)
- [Procedural textures must be baked](procedural-textures-need-bake.md)

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] - wspolne: linear, srgb, gamma
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: srgb, gamma, blender
- [[20260720-0940-dwa-niezalezne-bledy-gamma-w-jednym-modelu|"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw]] - wspolne: gamma, vertex-color
- [[20260720-0915-loft-nie-da-plaskiej-szyby|Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli]] - wspolne: vertex-color, blender
<!-- /POWIAZANE:auto -->
