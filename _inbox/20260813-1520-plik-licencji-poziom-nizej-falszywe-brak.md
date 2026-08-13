---
title: Szukanie licencji tylko w korzeniu repozytorium daje falszywe "brak licencji"
type: lesson
status: draft
confidence: high
verified: 2026-08-13
tags: [licencje, audyt, blender, dodatki, narzedzia, ryzyko-prawne]
date: 2026-08-13
project: GameDevOS
source: https://projects.blender.org/Mets/CloudRig
applies_to: [audyt-cudzego-kodu, dodatki-blendera, ocena-narzedzi]
severity: high
time_lost: ok. 2 dni opoznienia w ocenie narzedzia
---

# Plik licencji bywa poziom nizej, a "brak licencji" jest wyrokiem

## Objaw

Automat sprawdzajacy licencje odczytal korzen repozytorium `Mets/CloudRig`
(dodatek do Blendera do generowania rigow, opiekun Blender Studio) i znalazl
**dwanascie wpisow, w tym ani jednego pliku licencji**. Werdykt automatu brzmial
"BRAK licencji, formalnie nie wolno tego uzyc".

To bylo falszywe. Licencja jest: **doslowny GPL-3.0 o dlugosci 35 149 bajtow**
lezy w `CloudRig/LICENSE`, czyli **jeden poziom nizej**.

## Przyczyna

Dwie rzeczy naraz, obie strukturalne, nie przypadkowe.

**1. Dodatki do Blendera maja obudowe projektu w korzeniu, a wlasciwy dodatek
w podkatalogu.** W korzeniu stoi konfiguracja narzedzi, testy, dokumentacja
i wymagania. Sam dodatek, razem z `blender_manifest.toml` i licencja, siedzi
w katalogu nazwanym jak dodatek. Szukanie wylacznie w korzeniu **mija licencje
z definicji**, a nie przez pecha.

**2. Galaz domyslna nie zawsze nazywa sie `main`.** Na serwerze
`projects.blender.org` (Forgejo) jest to `master`. Co gorsza, adres surowy
z czlonem `main` **nie zwraca bledu**: oddaje tekst `Not found.` z kodem
**HTTP 200**. Automat, ktory patrzy na kod odpowiedzi, uzna to za sukces
i wczyta trzynastoznakowy "plik licencji".

## Rozwiazanie

1. **Nazwe galezi odczytuj z interfejsu programistycznego, nie zgaduj.**
   Pole `default_branch` w danych repozytorium.
2. **Gdy w korzeniu nie ma licencji, zejdz o jeden poziom** do podkatalogow,
   pomijajac katalogi zaczynajace sie od kropki oraz typowe katalogi bez
   licencji (`docs`, `tests`, `examples`, `translations`, `icons`, `assets`).
3. **Gdy licencja JEST w korzeniu, nie schodz nizej ani o katalog.** Inaczej
   zbierzesz licencje cudzych skladnikow i pomylisz je z licencja projektu.
4. **Odrzucaj tresc krotsza niz naglowek licencji.** To lapie odpowiedzi
   w rodzaju `Not found.` udajace plik.
5. Przy dodatkach Blendera czytaj **`blender_manifest.toml` obok pliku
   licencji** - bywa jedynym miejscem, gdzie licencja w ogole stoi (pole
   `license = ["SPDX:..."]`).

## Co NIE zadzialalo

- **Poleganie na kodzie odpowiedzi HTTP.** Forgejo oddaje 200 na nieistniejacy
  plik. Kod stanu klamal.
- **Zalozenie, ze galaz nazywa sie `main`.** Cala rodzina serwerow starszych
  niz 2020 rok trzyma `master`.
- **Czytanie etykiety licencji zamiast pliku.** Etykieta bywa zgadywana
  automatem i przy dodatkach spolecznosciowych myli sie regularnie.

## Dowod

Zmierzone 13.08.2026 z kontrolami:

- korzen `Mets/CloudRig`: 12 wpisow, zero plikow licencji
- `CloudRig/LICENSE` na galezi `master`: **HTTP 200, 35 149 bajtow**, pierwsze
  linie to `GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007`
- ten sam plik z czlonem `main` w adresie: **HTTP 200 i tresc `Not found.`**
- **kontrola pozytywna:** `blender/blender` ma licencje (`COPYING`) w korzeniu
  i musi dac **inny** wynik, czyli zejscie nizej nie ma nastapic
- **kontrola negatywna:** zmyslone repozytorium daje HTTP 404 i werdykt
  "nie udalo sie sprawdzic", a **nie** "brak licencji"

Poprawka zamknieta szescioma sprawdzianami; dzwignia porazki (zgadywanie
galezi zamiast odczytu) lamie trzy z nich.

## Czy to przeniesie sie na inny projekt

**Tak, i to szeroko.** Regula "brak pliku licencji = nie wolno uzyc" jest
poprawna i warto ja miec, ale **falszywy alarm na tej regule kosztuje tyle
samo, co przeoczenie**: dobre narzedzie wypada z rozwazan bez odwolania,
z tym samym werdyktem, co narzedzie faktycznie nas wykluczajace.

Dotyczy kazdego projektu, ktory ocenia cudzy kod przed uzyciem, niezaleznie
od silnika i gatunku gry. Struktura "obudowa w korzeniu, pakiet w podkatalogu"
jest powszechna takze poza Blenderem: paczki Pythona (`src/`), monorepozytoria,
wtyczki Unity w `Packages/`.

## Powiazane

- [[20260808-sprawdzian-ktory-nie-umie-pasc]]
- [[etykieta-licencji-to-nie-licencja]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260808-1120-zasoby-w-dodatkach-blendera-maja-byc-cc0|Zasoby w dodatkach Blendera maja byc na CC0, a nie na GPL dodatku]] - wspolne: dodatki, licencje, blender
- [[20260810-0930-migawka-zamiast-zapytania-gdy-kanal-nie-ma-daty-zmiany|Migawka zamiast zapytania, gdy kanal nie ma daty zmiany]] - wspolne: narzedzia, licencje
- [[20260727-1309-naprawiony-suwak-uniewaznia-strojenie|Naprawa suwaka, ktory po cichu klamal, uniewaznia CALE wczesniejsze strojenie]] - wspolne: narzedzia, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: licencje, blender
<!-- /POWIAZANE:auto -->
