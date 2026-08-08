---
title: QuadriFlow kasuje UV i wagi szkieletu, dopoki nie wlaczysz jednej flagi
type: lesson
status: verified
confidence: high
verified: '2026-08-05'
tags:
- blender
- retopologia
- low-poly
- skrypty
- pipeline
date: '2026-08-01'
project: GameDevOS
source: https://github.com/blender/blender/blob/main/source/blender/editors/object/object_remesh.cc
suggested-category: tools/lessons
applies_to:
- Blender 5.x
severity: high
time_lost: 'zero, zlapane pomiarem przed uzyciem w produkcji'
---

# QuadriFlow kasuje UV i wagi szkieletu, dopoki nie wlaczysz jednej flagi

## Objaw

Puszczasz automatyczna retopologie QuadriFlow na modelu, ktory ma juz rozwiniete
UV i przypisane wagi do kosci. Dostajesz ladna siatke czworokatna. Tekstury
przestaja sie trzymac modelu, a postac przy animacji rozpada sie, bo wagi
szkieletu sa wyzerowane.

## Przyczyna

Operator QuadriFlow ma parametr `preserve_attributes`, ktory jest **domyslnie
wylaczony**. Bez niego operacja buduje siatke od zera i nie przenosi zadnych
atrybutow: ani wspolrzednych UV, ani grup wierzcholkow trzymajacych wagi.

To nie jest blad. To domyslna wartosc, ktora ma sens przy modelowaniu od zera
i nie ma go zupelnie przy przerabianiu gotowego modelu. Parametr nie pojawia sie
w wiekszosci poradnikow, wiec latwo o nim nie wiedziec.

## Rozwiazanie

Jeden argument przy wywolaniu:

    bpy.ops.object.quadriflow_remesh(
        target_faces=1500,
        preserve_attributes=True,
    )

Zmierzone na siatce 62 976 trojkatow z rozwinieciem UV i jedna grupa wag:

| `preserve_attributes` | wynik | UV | wagi | czas |
|---|---|---|---|---|
| `False` (domyslnie) | 1490 czworokatow | **skasowane** | **skasowane** | 0,64 s |
| `True` | 1490 czworokatow | zachowane | zachowane | 0,65 s |

**Koszt wydajnosciowy jest zerowy.** Nie ma zadnego powodu, zeby tej flagi
nie wlaczac przy pracy na gotowym modelu.

## Co NIE zadzialalo i czego nie trzeba sie bac

- **Obawa, ze QuadriFlow nie dziala bezokienkowo, jest nieaktualna.** Sprawdzone:
  dziala w trybie `--background`. Zgloszenia o wysypywaniu sie przez `bpy`
  dotycza Blendera 3.5 do 4.1.
- **Obawa, ze wymaga siatki zamknietej (manifold), tez jest nieaktualna.**
  Sprawdzone trzy przypadki brzegowe: otwarta plaszczyzna bez objetosci, siatka
  ze zdublowana sciana wewnetrzna oraz 251 904 trojkaty. Wszystkie zwrocily
  `{'FINISHED'}`, ostatni w 3,46 sekundy.

## Gdy UV z generatora sa zle

Przy modelach z generatorow AI rozwiniecie UV czesto nadaje sie do wyrzucenia.
Wtedy zamiast `preserve_attributes` uzyj lancucha, ktory tez przechodzi w jednym
wywolaniu bezokienkowym:

1. QuadriFlow z `target_faces`
2. `bpy.ops.uv.smart_project()` - odbudowa UV od zera
3. modyfikator `DATA_TRANSFER` z `data_types_verts={'VGROUP_WEIGHTS'}`
   i `vert_mapping='POLYINTERP_NEAREST'`, poprzedzony
   `bpy.ops.object.datalayout_transfer()`

Wagi przechodza z dokladnoscia zmiennoprzecinkowa (0,7 zamienia sie w 0,6999999).

## Dowod

Pomiary wykonane lokalnie na Blenderze 5.2.0 LTS w trybie `--background`,
2026-08-01. To sa **liczby zmierzone u nas**, a nie przepisane z dokumentacji -
i dlatego ta lekcja jest warta wiecej niz strona podrecznika.

## Czy to przeniesie sie na inny projekt

Tak. To jest zachowanie **narzedzia**, nie projektu. Dotyczy kazdego potoku,
w ktorym model o duzej liczbie trojkatow trzeba sprowadzic do low poly
i zachowac przy tym cokolwiek poza sama geometria. Przy pracy skryptowej
i bezokienkowej to jest roznica miedzy dzialajacym potokiem a cichym psuciem
kazdego modelu.

## Powiazane

- [[MAPA-PIPELINE-BLENDER-UNITY]]
- [[MAPA-LOW-POLY]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: pipeline, low-poly, blender
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] - wspolne: retopologia, blender
- [[20260807-0900-normalne-nie-uspojniac-bezwarunkowo|20260807-0900-normalne-nie-uspojniac-bezwarunkowo]] - wspolne: pipeline, blender
- [[20260807-1155-skrypt-zarzadzany-blender-bezokienkowy|Skrypt zarzadzany - dyscyplina edycji Blendera bezokienkowo]] - wspolne: pipeline, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: low-poly, blender
- [[20260805-1815-rigify-ma-gotowe-szkielety-zwierzat|Blender ma w standardzie gotowe szkielety zwierzat (Rigify), zanim siegniesz po AI]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
