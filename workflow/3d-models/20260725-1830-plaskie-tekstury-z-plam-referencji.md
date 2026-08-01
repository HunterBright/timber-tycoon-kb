---
title: 'Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- blender
- tekstury
- generatory-3d
- retopologia
- bake
- referencja
- numpy
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru

## Problem

Model z generatora obraz->3D (TRELLIS.2, TripoSG, Hunyuan) po retopologii nie ma uzywalnej
tekstury. Rzutowanie referencji z przodu i tylu daje szwy i rozmycia na powierzchniach widzianych
stycznie (skronie, uszy, boki tulowia), bo tam zaden rzut nie widzi powierzchni.

Naturalny odruch - "pogrupuj piksele po kolorze i wypelnij plaskimi kolorami" - NIE DZIALA.

## Dlaczego grupowanie po kolorze zawodzi (4 sprawdzone warianty)

| podejscie | wynik |
|---|---|
| k-srednich na surowym kolorze | odblask elementu ladauje w innej grupie niz jego cien: czapka dwukolorowa, laty na spodniach |
| kolor + pozycja na ciele | laty znikaja, ale pozycja zjada male detale (oczy, brwi) |
| sam odcien (kat barwy) | skora, kamizelka, pas, buty i rekawice maja TEN SAM odcien: jeden kolor pokryl 64% modelu |
| scalanie "odblask + baza" po odcieniu i wspolnym miejscu | LAWINA scalen - zacieniony fragment skory ma nasycenie kamizelki i robi pomost; 75% modelu w jednym kolorze |

Wniosek: kolor piksela nie niesie informacji, do KTOREGO elementu ubioru nalezy.

## Wzorzec, ktory dziala

1. **Podziel REFERENCJE na plamy, nie model.** Wewnatrz elementu kolor zmienia sie lagodnie
   (cieniowanie), na granicy elementow skacze. Rozrost obszarow po lokalnym skoku koloru daje
   plamy = czapka, wlosy, twarz, oczy, kamizelka, pas, spodnie, buty.
2. **Rozrost musi byc OSTRY + osobne laczenie z limitem rozrzutu.** Sam lagodny prog przecieka:
   granat czapki -> miekki cien pod daszkiem -> skora czola to lancuszek malych krokow, ktory
   scala czapke z twarza. Prog 0.028 (w przestrzeni sqrt) + laczenie sasiadow do 0.045, dopoki
   laczna rozpietosc plamy < 0.13.
3. **Jedna plama = jeden kolor** (mediana z przedzialu 40-75 percentyla jasnosci - bez odblaskow
   i cieni).
4. **Przod i tyl zszywaj po GEOMETRII, nie po kolorze**: plama z tylu przyjmuje kolor plamy z
   przodu lezacej po drugiej stronie sylwetki (glosowanie najblizszych sasiadow w 3D). Straznik
   stosunku jasnosci < 2 chroni przed "wlosy przyjmuja kolor twarzy".
5. **Lustrzane polowki wyrownuj osobno**: referencja oswietlona z jednej strony daje dwa odcienie
   tego samego elementu (lewy i prawy panel kamizelki). Partnera szukaj przez odbicie polozenia
   w osi ciala - nigdy po samym kolorze.
6. **Pas styczny wypelniaj najblizszymi sasiadami (KD-drzewo), nie wiekszoscia w kratce** - kratka
   miesza sasiednie elementy i robi schodki.
7. **Czego referencja nie umie, dopisz regulami z geometrii.** Model z generatora bywa
   NIEZGODNY z referencja (tu: czapka wieksza niz na obrazku, wiec jej powierzchnia trafiala w
   piksele czola). Czapke wyznacza sie ksztaltem: daszek = najdalszy wystep do przodu przy pelnej
   szerokosci glowy, wszystko powyżej to czapka.

## Pulapki techniczne

- **Wypiek danych do obrazka 8-bitowego psuje wartosci.** Pozycja punktu zapisana jako kolor
  przechodzi przez krzywa sRGB. Algorytmy "po sasiedztwie" tego NIE ZAUWAZA (zniekształcenie jest
  monotoniczne), ale kazdy wzor liczony na wspolrzednych daje bzdury. Wypiekac do obrazka
  zmiennoprzecinkowego z przestrzenia 'Non-Color'.
- **Ta sama krzywa rozjasnia gotowa teksture** (podwojne kodowanie przy zapisie).
- **Nie zakladaj ludzkich proporcji przy pomiarach.** Postac kreskowkowa moze miec glowe = 30%
  wzrostu, siedzaca praktycznie na ramionach, BEZ zwezenia szyi. Szukanie szyi w "ludzkim"
  zakresie wysokosci zwraca bzdure.
- **Ochrona detali twarzy musi byc kierunkowa.** Regula "nie ruszaj ciemnych etykiet" chroni oczy
  i brwi, ale broni tez ciemnych smieci na boku i tyle glowy. Ograniczyc do powierzchni
  zwroconych do przodu.

## Kontrole, ktore oplacily sie najbardziej

Skrypt sam przerywal prace, gdy: (a) wzor rzutu nie zgadzal sie z realnie wypieczonym kolorem
(sredni blad 0.22 zamiast < 0.06), (b) plama czubka glowy nie byla niebieskawa (czyli nie byla
czapka), (c) nie znalazl koloru skory na twarzy. Kazde przerwanie wskazalo prawdziwa przyczyne
zamiast wypuscic zepsuta teksture. Uklad wspolrzednych i przestrzen koloru referencji nie sa
zgadywane - skrypt sprawdza wszystkie warianty i wybiera ten, ktory zgadza sie z pomiarem.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: generatory-3d, blender
- [[20260801-1130-quadriflow-kasuje-uv-i-wagi-bez-jednej-flagi|QuadriFlow kasuje UV i wagi szkieletu, dopoki nie wlaczysz jednej flagi]] - wspolne: retopologia, blender
- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] - wspolne: bake, blender
- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] - wspolne: bake, blender
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] - wspolne: bake, blender
- [[20260727-1921-kamera-z-siatki-podlogi-nie-z-sylwetki|Kamerę odtwarzaj z regularnej struktury sceny, nie z sylwetki modelu]] - wspolne: referencja, blender
<!-- /POWIAZANE:auto -->
