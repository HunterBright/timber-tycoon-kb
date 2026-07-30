---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, particles, shuriken, transparency, sorting, water, urp]
date: 2026-07-18
status: draft
---

# Czasteczki "dzialaja, ale ich nie widac" - trzy niezalezne przyczyny przy wodzie

## Kontekst
Mgielka + rozbryzg u podstawy wodospadu (Shuriken, URP Particles/Unlit alpha fade). particleCount > 0,
materialy poprawne, zero magenty - a na zrzutach z gry efektu nie widac. 4 iteracje diagnozy.

## Lekcja (3 przyczyny, kazda osobno wystarczy, by "zjesc" efekt)

1. **Emiter pod tafla wody.** Pozycja liczona z dolu OBRYSU siatki (bounds.min.y) - a dol obrysu
   kurtyny wodospadu lezal ~0.9 m PONIZEJ linii styku z woda. Czasteczki zyly pod woda.
   FIX: pozycja z CENTROIDU dolnych wierzcholkow siatki (verts z y < min+2m), nie z bounds.

2. **Sortowanie przezroczystosci vs wielka tafla.** Unity sortuje transparenty po srodku obrysu
   renderera; srodek wielkiej tafli rzeki/jeziora wypada "gdzie popadnie", wiec woda potrafi
   rysowac sie PO czasteczkach i je zakryc. FIX: material czasteczek renderQueue = 3001
   (Transparent+1) - czasteczki nad woda zawsze rysuja sie po wodzie.

3. **Bialy efekt na bialym tle.** Mgielka w linii kurtyny zlewa sie z jej piana. FIX: wysunac
   emiter 1.5 m PRZED kurtyne (nad ciemniejsza tafle) + lekki blekitny odcien (0.87, 0.94, 1.0).

## Jak diagnozowac
Nie na oko w Edytorze: ps.Simulate(5f) + tymczasowa kamera + RenderTexture -> PNG w batchmode,
z 4 stron + zblizenia, plus log particleCount. Kazda hipoteza = jeden zrzut, zero zgadywania.
