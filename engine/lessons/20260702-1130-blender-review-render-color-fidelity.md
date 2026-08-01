---
title: 'Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- blender
- render
- color-management
- agx
- review
- low-poly
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"

## Problem
Rendery kontrolne modeli (6 nowych gatunków drzew) wychodziły wyblakłe - oliwkowa szarość
zamiast zieleni z zaakceptowanej karty kolorów. Wyglądało to na błąd materiałów, ale wartości
Base Color były poprawne.

## Przyczyna
Domyślny view transform Blendera (AgX, wcześniej Filmic) celowo desaturuje i spłaszcza kolory
(filmowy tone mapping). Do finalnego renderu artystycznego OK, ale do WERYFIKACJI wierności
kolorów względem specyfikacji - mylące. Dodatkowo mocne słońce + jasny world rozjaśniają
flat-color materiały low-poly.

## Rozwiązanie
W skryptach renderów kontrolnych: `scene.view_settings.view_transform = 'Standard'`
(+ look None, exposure 0, gamma 1) i stonowane oświetlenie (sun ~2.0, world ~0.5).
Wtedy piksele renderu odpowiadają wartościom Base Color i można porównywać z kartą kolorów 1:1.

## Reguła
Render „czy kolory się zgadzają" ≠ render „czy ładnie wygląda". Do pierwszego zawsze Standard.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] - wspolne: color-management, blender
- [[20260720-1410-ortho-comparison-render-hides-occlusion|Rzut prostokątny w ujęciu porównawczym potrafi pokazać kilka obiektów nałożonych na siebie i wyglądać jak jeden poprawny obiekt]] - wspolne: render, blender
- [[20260727-2145-sprawdzaj-czytnik-obrazu-renderem-wlasnego-modelu|Czytnik obrazu sprawdzaj renderem własnego modelu, nie obrazkiem, który sam sobie narysowałeś]] - wspolne: render, blender
- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] - wspolne: low-poly, blender
- [[20260702-1400-bark-atlas-vs-tile-strategy|Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową]] - wspolne: low-poly, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
