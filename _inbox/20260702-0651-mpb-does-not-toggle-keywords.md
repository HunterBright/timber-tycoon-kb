---
type: lesson
project: Timber Tycoon
severity: medium
suggested-category: engine/lessons
tags: [unity, urp, material-property-block, emission, shader-keywords]
date: 2026-07-02
status: draft
---

# Lekcja: MaterialPropertyBlock NIE włącza keywordów shadera (emisja niewidoczna)

## Symptom
Sterowanie `_EmissionColor` przez MaterialPropertyBlock nie daje żadnego efektu na
materiale URP/Lit, mimo poprawnych wartości HDR.

## Przyczyna
MPB nadpisuje WARTOŚCI właściwości, ale nie przełącza SHADER KEYWORDS. Jeśli materiał
nie ma włączonego `_EMISSION` (checkbox Emission w Inspektorze / `EnableKeyword("_EMISSION")`
na assecie materiału), shader w ogóle nie próbkuje `_EmissionColor` — MPB pisze w próżnię.

## Fix
Keyword musi być włączony NA MATERIALE (asset), a MPB steruje tylko kolorem/intensywnością:
- materiał emissive przygotowany w pipeline (u nas: `Mat_Drum_Interior_Emissive` ma
  `_EMISSION` włączone od importu) → runtime MPB `_BaseColor` + `_EmissionColor` działa od razu;
- alternatywa dla per-instance: `renderer.material` (instancja) + `EnableKeyword` raz przy inicie
  (tak robią guziki konsoli), kosztem instancji materiału.

## Zwalidowane
FertilizerMaker Faza 5: wnętrze bębna (szary/niebieski/złoty/czerwień + gradienty) w 100%
przez MPB na wcześniej-emissive materiale; potwierdzone w Play Mode 2026-07-02.
