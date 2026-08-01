---
title: Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- shader
- materials
- serialization
- urp
- backwards-compatibility
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach

## Kontekst
Rozszerzanie custom shadera (VertexColorLit) o tint koloru dla nowych koron drzew.
Plan: dodać `_BaseColor` z defaultem białym = "backwards compatible, stare materiały
bez zmian".

## Lekcja
Unity NIE usuwa z pliku .mat wartości properties, których aktualny shader nie ma.
Jeśli materiał kiedyś używał innego shadera (np. URP/Lit), jego stary `_BaseColor`
siedzi zapisany w YAML "na zapas". Gdy shader ZYSKA property o tej samej nazwie,
stara wartość natychmiast się aktywuje - default białego NIE chroni takich materiałów.

Case: `Mat_Debris_Dirt.mat` (VFX kopania pniaka) miał zapisany brązowy `_BaseColor`
z poprzedniego shadera. Dodanie `_BaseColor` do VertexColorLit przyciemniłoby efekt
kopania (vertex color × brąz), mimo defaultu (1,1,1,1).

## Reguła praktyczna
Przed dodaniem property do współdzielonego shadera:
1. `grep` po WSZYSTKICH .mat używających tego shadera (po GUID shadera) w poszukiwaniu
   nazwy property, którą chcesz dodać.
2. Jeśli JAKIKOLWIEK materiał ma ją zapisaną z nie-defaultową wartością → nazwij
   property unikalnie (np. `_LeafTint` zamiast `_BaseColor`). Unikalna nazwa =
   gwarancja, że default zadziała wszędzie.
3. Weryfikacja jest tania: `grep -l <guid-shadera> *.mat`, potem `grep <property>` w wynikach.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: shader, materials, urp
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] - wspolne: shader, materials, urp
- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] - wspolne: shader, materials
- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: shader, urp
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] - wspolne: shader, urp
- [[20260728-0910-urp-jednostronne-kartki-listowia-czarne|Jednostronne kartki listowia na URP/Lit wychodza CZARNE]] - wspolne: shader, urp
<!-- /POWIAZANE:auto -->
