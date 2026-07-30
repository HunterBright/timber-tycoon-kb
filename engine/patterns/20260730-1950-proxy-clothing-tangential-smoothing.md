---
title: 'Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- blender
- clothing
- proxy
- laplacian
- smoothing
- low-poly
- shrinkage
- rigging
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu

## Kontekst

Ubrania generowane jako kopie ścianek ciała z offsetem (proxy clothing) na
kanciastym, low-poly ciele. Wymaganie: ubrania gładkie ("nie kanciaste"),
riggowalne (wagi dziedziczone z ciała).

## Problem

Zwykłe wygładzanie laplasjanowe (v = mix(v, śr. sąsiadów)) na rurze
(nogawka, rękaw) OBKURCZA ją promieniowo - tkanina zapada się pod grzbiety
krawędzi kanciastego ciała i skóra przebija cienkimi smugami. Klamry
per-wierzchołek tego nie łapią (grzbiet przebija przez WNĘTRZE ścianki).

Ślepe uliczki (sprawdzone, nie działają):
1. Większy margines klamry per wierzchołek - smugi zostają (wnętrza ścianek).
2. Próbkowanie środków ścianek i krawędzi - zbiega za wolno, smugi maleją
   ale nie znikają.
3. Rzut na powierzchnię offsetową (najbliższa ścianka ciała + stały odstęp)
   - przy wewnętrznych udach/kostkach przypina tkaninę JEDNEJ nogawki do
   DRUGIEJ nogi (najbliższa powierzchnia!) i robi się gorzej; dodatkowo
   tkanina obciska mięśnie jak spandex.

## Rozwiązanie

Wygładzanie WYŁĄCZNIE STYCZNE: od wektora do średniej sąsiadów odejmij
składową wzdłuż normalnej wierzchołka:

```python
d = (śr_sąsiadów - v.co)
d -= v.normal * d.dot(v.normal)
v.co += d * 0.55
```

Wierzchołki rozprowadzają się równomiernie PO powierzchni (miękka
dystrybucja), ale rura nie traci objętości - zero przebić. Normalne
aktualizować co iterację (`bm.normal_update()`), brzegi przypiąć.

Efekt gładkości domyka **smooth shading z ostrymi krawędziami tylko na
brzegach i załamaniach >50 st.** (zero kosztu geometrii).

## Bonus: rąbek po wagach, nie po wysokości

Dolna granica ubrania wybieranego warunkiem `waga(kość) > próg AND z > próg`
kończy się po NIERÓWNEJ granicy wag (schodki na ręcznie malowanym ciele),
gdy wagi wygasają PONAD progiem wysokości. Objaw: zmiana progu z nic nie
daje. Fix: połączyć wagi sąsiednich kości (np. Hips+UpperLeg), żeby
granicę wyznaczała wysokość.

## Weryfikacja

Sonda przebić per stan suwaka (klucze sylwetek) + test deformacji po
eksporcie FBX (obrót kości → wierzchołki ubrania muszą się ruszyć).
