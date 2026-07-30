---
title: 'Transfer morfa przez proxy: przycięte barycentryki RWĄ siatkę, rzut na płaszczyznę + wygładzenie działa'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- proxy-mesh
- shape-keys
- makehuman
- binding
- morph-transfer
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Transfer morfa przez proxy: przycięte barycentryki RWĄ siatkę, rzut na płaszczyznę + wygładzenie działa

## Kontekst

Przenoszenie kształtów sylwetek (targety MakeHumana) na własną siatkę low poly
(765 wierzch.) metodą proxy: każdy wierzchołek przypięty do najbliższego trójkąta
siatki źródłowej, po morfie źródła odtwarzany z zapamiętanych współrzędnych.

## Problem

Pierwsza wersja: `BVHTree.find_nearest` → punkt najbliższy NA trójkącie
(przycięty do krawędzi) + wektor odsunięcia w układzie lokalnym trójkąta
(e0, e1, n). Efekt po morfie: **uda rozerwane na strzępy** (maks. delta 202 mm
przy targecie mięśni, który realnie rusza ~30 mm).

Mechanizm: dla wierzchołków DALEKO od powierzchni źródła (u nas nogi ~55 mm,
podwójne warstwy mankietów) punkt najbliższy ląduje na KRAWĘDZI trójkąta,
odsunięcie ma dużą składową styczną, a morf obraca trójkąt → składowa styczna
wymachuje wierzchołkiem. Sąsiednie wierzchołki przypięte do RÓŻNYCH trójkątów
wymachują w różne strony = rozdarcie.

## Rozwiązanie (zmierzone: maks. delta 202→62 mm, rendery czyste)

1. **Barycentryki z rzutu na PŁASZCZYZNĘ trójkąta, BEZ przycinania do [0,1]** -
   wagi ekstrapolowane poza trójkąt są funkcją ciągłą między sąsiednimi
   trójkątami, więc sąsiedzi jadą spójnie.
2. **Odsunięcie wyłącznie wzdłuż normalnej** (podpisana odległość do płaszczyzny).
3. **Wygładzenie pola delt po sąsiedztwie siatki docelowej** (3 iteracje,
   50% średniej sąsiadów) - tłumi resztkowe wymachy pojedynczych wierzchołków.

## Reguła

Przy każdym wiązaniu proxy/wrap (ubrania MakeHumana, Surface Deform bez fallbacku,
własne skrypty): jeśli siatka docelowa może leżeć DALEKO od źródła, nie wolno
używać przyciętego punktu najbliższego z odsunięciem stycznym. Rzut na płaszczyznę
+ nieprzycięte barycentryki + offset normalny, opcjonalnie wygładzenie delt.

## Dowód

`_BlenderScripts/kerf_generator/s2_sylwetki.py` (wersja przed/po w gicie),
rendery: `_BlenderOutputs/KerfGenerator/S2/` (pierwsza wersja vs druga).
