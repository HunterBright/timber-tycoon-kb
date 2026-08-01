---
title: 'Lekcja: MaterialPropertyBlock NIE włącza keywordów shadera (emisja niewidoczna)'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- materialpropertyblock
- emission
- shader-keywords
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Lekcja: MaterialPropertyBlock NIE włącza keywordów shadera (emisja niewidoczna)

## Symptom
Sterowanie `_EmissionColor` przez MaterialPropertyBlock nie daje żadnego efektu na
materiale URP/Lit, mimo poprawnych wartości HDR.

## Przyczyna
MPB nadpisuje WARTOŚCI właściwości, ale nie przełącza SHADER KEYWORDS. Jeśli materiał
nie ma włączonego `_EMISSION` (checkbox Emission w Inspektorze / `EnableKeyword("_EMISSION")`
na assecie materiału), shader w ogóle nie próbkuje `_EmissionColor` - MPB pisze w próżnię.

## Fix
Keyword musi być włączony NA MATERIALE (asset), a MPB steruje tylko kolorem/intensywnością:
- materiał emissive przygotowany w pipeline (u nas: `Mat_Drum_Interior_Emissive` ma
  `_EMISSION` włączone od importu) → runtime MPB `_BaseColor` + `_EmissionColor` działa od razu;
- alternatywa dla per-instance: `renderer.material` (instancja) + `EnableKeyword` raz przy inicie
  (tak robią guziki konsoli), kosztem instancji materiału.

## Zwalidowane
FertilizerMaker Faza 5: wnętrze bębna (szary/niebieski/złoty/czerwień + gradienty) w 100%
przez MPB na wcześniej-emissive materiale; potwierdzone w Play Mode 2026-07-02.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260720-0940-dwa-niezalezne-bledy-gamma-w-jednym-modelu|"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw]] - wspolne: materialpropertyblock, urp
- [[20260721-1845-tint-palette-on-textured-mesh-neutralize-average|Paleta kolorow na TEKSTUROWANEJ siatce: dziel tint przez sredni kolor tekstury]] - wspolne: materialpropertyblock, urp
<!-- /POWIAZANE:auto -->
