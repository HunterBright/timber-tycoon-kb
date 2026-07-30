---
title: Proceduralny rozsiew dekoracji musi wykluczac pozycje obiektow interaktywnych
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Timber_Tycoon
tags:
- unity
- procedural
- scatter
- flora
- gpu-instancing
- level-design
applies_to: []
source: ''
severity: low
promoted: '2026-07-30'
---

# Proceduralny rozsiew dekoracji musi wykluczac pozycje obiektow interaktywnych

## Kontekst
Trawa mapy = ~98k kartek GPU-instanced, pozycje zapisane w binarnym blobie bake'owanym
przez narzedzie scatter. Rozsiew nie omijal pozycji drzew.

## Problem
Dekoracja nachodzi na obiekty, ktore ZMIENIAJA STAN w gameplayu: drzewo -> pniak -> dolek
sadzenia. Kartka trawy stojaca "w pniu" jest niewidoczna przy pelnym drzewie, ale po scieciu
i wykopaniu pniaka wyrasta ze srodka dolka. Blad ujawnia sie dopiero po sekwencji gameplayowej,
nie w edytorze.

## Rozwiazanie / regula
1. Narzedzie scatter powinno przyjmowac zbior stref wykluczenia: pozycje wszystkich obiektow
   interaktywnych/zmiennych (drzewa, spawnpointy, budynki) + promien pokrywajacy ich NAJWIEKSZA
   forme (u nas: podkladka miejsca sadzenia 1x1 m -> promien 0.8 m > polprzekatna 0.71 m).
2. Gdy blob juz istnieje i jest zaakceptowany wizualnie - taniej PRZEFILTROWAC blob
   (usunac instancje w promieniu od kotwic) niz re-scatterowac cala mape. Bounds komorek
   spatial-grid mozna zostawic oryginalne (nadzbior pozostalych instancji = culling dalej poprawny).
3. Weryfikacja mechaniczna: skrypt liczacy instancje w promieniu X od kazdej kotwicy
   (przed: 52 w 0.5 m; po: 0 w 0.8 m) - dowod zamiast ogladania mapy.

## Uwaga na przyszlosc
Wykluczenie musi trafic do NARZEDZIA scatter, nie tylko do jednorazowego filtra - inaczej
kazdy re-bake przywraca problem.
