---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [blender, bmesh, ubrania, low-poly, bisect_plane, krawedzie]
date: 2026-07-31
status: draft
---

# Równe krawędzie ubrań z kopii siatki: cięcie płaszczyzną zamiast selekcji ścianek

## Problem
Ubranie budowane jako kopia ścianek ciała (wybór per wierzchołek: wagi kości
+ zakresy wysokości) ma brzegi będące zygzakiem Z NATURY - granica podąża za
topologią ręcznie rzeźbionego, niesymetrycznego ciała. Rąbki, pasy i mankiety
wyglądają jak poszarpane. Wygładzanie brzegu (średnia sąsiadów wzdłuż brzegu)
tylko łagodzi ząbki, nie robi prostej linii; ściąganie brzegu do mediany
wysokości psuje mapowanie kluczy kształtu.

## Wzorzec
1. Selekcję ścianek robić Z ZAPASEM (progi ~2-3 cm poza docelową granicę;
   ścianka przecinająca przyszłe cięcie musi być CAŁA w selekcji, inaczej
   krawędź ma dziury).
2. Po skopiowaniu i dogęszczeniu ciąć `bmesh.ops.bisect_plane` na zadanych
   wysokościach (rąbki, pasy, mankiety) i szerokościach (boki karczka,
   szelki) - pary symetrycznych płaszczyzn ±x dają symetrię mimo
   niesymetrycznego ciała.
3. Po cięciach usunąć ścianki poza regionem testem ŚRODKA ścianki
   (`calc_center_median`), z wagami z warstwy deform (bisect je
   interpoluje). Każda granica regionu MUSI pokrywać się z płaszczyzną
   cięcia - wtedy krawędź jest prosta.
4. Ciąć PRZED mapowaniem źródeł i offsetem po normalnych - nowe wierzchołki
   z cięcia leżą na powierzchni ciała jak reszta.

## Ograniczenia (sprawdzone boleśnie)
- Granice będące KRZYWĄ (obwód szyi) nie dają się zamknąć płaszczyznami |x| -
  pierścień pęka na łuki, które sprzątanie wysp kasuje. Elementy obwodowe
  (pasek na szyję, opaski) budować PROCEDURALNIE z pomiaru promienia skóry
  (funkcja podparcia per kierunek), jak kopułę kasku.
- Sprzątanie małych wysp loguj per wyspa (liczba wierzchołków + zakres z) -
  niema kasacja 300 wierzchołków wyglądała jak "brak paska" bez śladu.
- Regiony rozłączne w pionie (karczek nad pasem) muszą ZACHODZIĆ na wspólną
  płaszczyznę cięcia, inaczej fragmenty są osobnymi wyspami.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw|20260801-0700-adr-npc-od-zera-zamiast-solvera-warstw]] - wspolne: ubrania, low-poly
- [[20260801-0500-gestszy-pomiar-odslania-dlug|20260801-0500-gestszy-pomiar-odslania-dlug]] - wspolne: ubrania, blender
- [[20260730-1217-bone-side-names-vs-axis-sign|Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)]] - wspolne: bmesh, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: low-poly, blender
- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] - wspolne: low-poly, blender
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
