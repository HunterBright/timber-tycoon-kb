---
title: LineRenderer lezacy plasko na powierzchni znika, bo material jest jednostronny
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- linerenderer
- backface-culling
- material
- editor-vs-runtime
- bakemesh
applies_to:
- unity6
- urp
source: ''
severity: medium
time_lost: ~40 min
promoted: '2026-07-30'
---

# LineRenderer lezacy plasko na powierzchni znika, bo material jest jednostronny

## Problem
Cienka linia (prowadnica ciecia) miala lezec plasko NA licu deski. Po zbudowaniu
geometrii wszystko wygladalo poprawnie w liczbach, ale na renderze deska byla CZYSTA -
linii nie bylo widac w ogole. Zadnego bledu, zadnego ostrzezenia, zaden log nie krzyczal.

Diagnostyka mylila dwukrotnie:
1. Pierwsze podejrzenie: zle pozycje albo zerowa dlugosc. Zrzut `LineRenderer.BakeMesh`
   pokazal poprawna bryle 0,450 x 0,004 x 0,000 m i poprawny srodek - geometria byla OK.
2. Drugie podejrzenie: pusta siatka. `baked.vertexCount` = 4, `triangles` = 2 - tez OK.

## Root cause
Dwie NIEZALEZNE przyczyny, ktore dawaly ten sam objaw:

**(a) Jednostronny material.** Wstazka LineRenderera to plaski, jednostronny quad.
Material na shaderze URP/Unlit ma domyslnie `_Cull = Back` (2). Przy `alignment =
LineAlignment.TransformZ` wstazka patrzy w strone lokalnej osi Z transformu - jesli ta os
jest odwrocona wzgledem kamery, widac TYL quada, czyli nic. Powierzchnia pod spodem
renderuje sie normalnie, wiec efekt to "linii nie ma", a nie "linia jest czarna".

**(b) Podglad w edytorze bez klatek.** LineRenderer buduje geometrie w petli klatek.
Skrypt edytorowy, ktory tylko ustawia pozycje i wola `camera.Render()`, nigdy tej petli
nie przepuszcza - linia renderuje sie jako pojedyncze plamki albo wcale. To dotyczy TYLKO
edytora/podgladow, nie gry.

## Solution
- **Material linii ZAWSZE dwustronny**: `mat.SetFloat("_Cull", 0f)` na assecie. Dla
  wstazki o szerokosci kilku milimetrow drugi przebieg nic nie kosztuje, a usuwa cala klase
  bledow "widac tylko z jednej strony".
- **Wstazka plasko w powierzchni**: `alignment = LineAlignment.TransformZ` + dziecko obrocone
  `Euler(-90,0,0)`, zeby lokalna os Z linii wskazywala w gore. Domyslne `LineAlignment.View`
  obraca wstazke do kamery i linia czyta sie jak naklejka doklejona NAD powierzchnia,
  nie jak rysa W powierzchni.
- **Podglad w edytorze**: `lr.BakeMesh(mesh, cam, false)` i wyrenderowanie zrzutu jako zwyklej
  siatki. Zrzut jest w PRZESTRZENI LOKALNEJ linii (gdy `useWorldSpace = false`), wiec obiekt
  z siatka wystarczy podpiac pod transform linii z zerowym offsetem.
- **Prosvit nad powierzchnia**: 1-2 mm. Zero = migotanie (z-fighting), 2 cm = kreska
  wyraznie lewituje. Wysokosc trzymac jako wartosc konfiguracyjna, nie stala w kodzie.

## What didn't work
- Zgadywanie z samych liczb. Bryla i liczba trojkatow byly poprawne przy OBU przyczynach,
  wiec zadna z tych metryk nie odrozniala "linia jest" od "linii nie widac".
- Ogladanie renderu pod plaskim katem: odbicie nieba (Fresnel) rozjasnia lico tak mocno,
  ze ciemne materialy wygladaja na jasnoszare i ocena koloru jest bezwartosciowa. Kadr
  diagnostyczny musi byc STROMY albo prostopadly.
- Ocenianie kierunku slojow "na oko" z kadru perspektywicznego. Rozstrzyga dopiero rzut
  z gory z JAWNIE ustawiona rotacja kamery (`Euler(90,0,0)`), bo `LookAt` pionowo w dol daje
  losowy obrot kadru.

## Transferability
Dotyczy kazdego projektu Unity, w ktorym LineRenderer/TrailRenderer ma lezec na powierzchni:
sciezki, prowadnice, obrysy, slady opon, znaczniki zasiegu. Punkt (b) dotyczy kazdego
narzedzia edytorowego renderujacego podglad przez `camera.Render()` bez odpalania gry.

## Related
- [[fbx-binary-overwrite-corrupts-bindposes]]
- [[flatten-must-be-baked-into-geometry-when-code-forces-uniform-scale]]
