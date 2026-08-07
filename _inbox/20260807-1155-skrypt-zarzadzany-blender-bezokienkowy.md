---
title: Skrypt zarzadzany - dyscyplina edycji Blendera bezokienkowo
type: pattern
status: draft
confidence: medium
verified:
tags: [blender, headless, automatyzacja, agent, idempotentnosc, pipeline]
date: 2026-08-07
project: Kerf - Sawmill Tycoon
source: https://github.com/czlonkowski/blender-web-3d-skill (plugins/blender-web-3d/skills/blender-web-3d/, MIT)
applies_to: [blender-4.x, blender-5.x, claude-code]
suggested-category: pipeline/patterns
---

# Skrypt zarzadzany: dyscyplina edycji Blendera bezokienkowo

## Kiedy stosowac

Zawsze, gdy model zmienia **agent**, a nie czlowiek klikajacy w oknie. Czyli
przy calej naszej pracy w `blender --background`.

Problem, ktory to rozwiazuje: agent uruchamia skrypt dwa razy i robi to samo
dwa razy; albo uruchamia go na niewlasciwym pliku; albo cos psuje i nie wiadomo
kiedy, bo nie ma do czego wrocic.

## Kroki

1. **Kopiuj do przodu, nie edytuj wstecz.** Jedna wersja pliku `.blend` na jedna
   sensowna zmiane. Poprzedniej wersji nie rusza sie nigdy.
2. **Kazdy skrypt pilnuje, ze trafil na wlasciwy plik.** Porownanie
   `bpy.data.filepath` ze sciezka docelowa, a przy niezgodnosci twardy blad.
3. **Kazdy skrypt jest idempotentny przez znacznik w scenie.** Wlasciwosc
   niestandardowa sceny trzyma numer wersji; jesli jest juz ustawiona, skrypt
   konczy sie komunikatem „juz zastosowane" i **niczego nie robi**.
4. **Kazdy skrypt sam sie sprawdza** i wypisuje wynik w formacie maszynowym
   (co usunieto, co zbudowano), zeby agent mial co czytac zamiast zgadywac.
5. **Weryfikacja obrazem, nie logiem.** Po zmianie render z dwoch do czterech
   kamer i **obejrzenie go**. Bledy geometrii sa oczywiste na obrazku
   i niewidoczne w logu.

## Dlaczego to dziala

Trzy zabezpieczenia odpowiadaja trzem trybom awarii pracy agentowej:

- **zly plik** - lapie straznik sciezki
- **podwojne wykonanie** - lapie znacznik wersji
- **cicha porazka** - lapie sprawdzenie i render

Wersjonowany lancuch plikow daje czwarta rzecz: **mozliwosc przepolowienia**.
Gdy cos sie zepsulo, a nie wiadomo kiedy, wraca sie do wersji, w ktorej bylo
dobrze, zamiast odtwarzac wszystko od zera.

## Koszty i kompromisy

- **Zajmuje miejsce na dysku.** Kazda wersja to pelny plik `.blend`.
- Wymaga dyscypliny nazewnictwa od pierwszego dnia; wprowadzone w polowie
  projektu daje polowe korzysci.
- Znacznik wersji w scenie jest **niewidoczny dla czlowieka** pracujacego w oknie
  Blendera i latwo o nim zapomniec przy recznej edycji.

## Warianty

Ten sam skrypt da sie napisac tak, zeby dzialal **i bezokienkowo, i przez most
MCP**. Warunki: straznik opiera sie na `bpy.data.filepath`, a nie na stanie okna;
zamiast operatorow wymagajacych kontekstu widoku 3D uzywa sie
`bpy.data.meshes.new_from_object(obj.evaluated_get(depsgraph))`; calosc jest
idempotentna. Uwaga praktyczna z tego samego zrodla: **dodatek mostu MCP
w trybie `--background` w ogole nie startuje** - wypisuje ostrzezenie i lezy
bezczynnie. Most sluzy do pracy na zywo z podgladem, potok idzie bezokienkowo.

## Dowod, ze zadzialalo u nas

**Jeszcze nie - to jest wpis do sprawdzenia.** Cudzy dowod: okolo trzydziestu
wersjonowanych skryptow edycji na wydanym projekcie.

Nasza wlasna przeslanka, ze to dobra droga, jest jednak mocna: **te same trzy
warunki (tryb probny, idempotentnosc, udowodniony tryb porazki) narzucamy sobie
przy kazdym narzedziu w `tools\`** i za kazdym razem wylapaly blad, ktorego
inaczej bysmy nie zobaczyli. To jest ten sam wzorzec przeniesiony na pliki
`.blend`.

## Powiazane

- [[MAPA-PIPELINE-BLENDER-UNITY]]
- [[20260807-1140-laczenie-siatek-a-animowany-przodek]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260805-1520-przedawnienie-wiedzy-jest-funkcja-typu-wpisu|Przedawnienie wpisu jest funkcja jego TYPU, nie uplywu czasu]] - wspolne: agent, automatyzacja
- [[ANALIZA-ROZMOWY|Analiza rozmowy o automatyzacji pipeline'u - mocne i słabe strony]] - wspolne: automatyzacja, pipeline
- [[DZWIGNIA-UNITY-CLI|Dźwignia, która stoi nieużywana - własne komendy w Unity CLI]] - wspolne: automatyzacja, pipeline
- [[20260807-1140-laczenie-siatek-a-animowany-przodek|Laczenie siatek pod wywolania rysowania a animowany przodek]] - wspolne: headless, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: pipeline, blender
- [[blender-headless-python-generation|Blender Headless Python Script Generation]] - wspolne: headless, blender
<!-- /POWIAZANE:auto -->
