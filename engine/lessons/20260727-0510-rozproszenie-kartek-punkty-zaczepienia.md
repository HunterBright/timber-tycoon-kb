---
title: 'Rozsypywanie kartek po szkielecie: licz NOSNIKI, nie kartki'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- proceduralne-generowanie
- blender
- alpha-cards
- listowie
- optymalizacja
- pomiar
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Rozsypywanie kartek po szkielecie: licz NOSNIKI, nie kartki

## Objaw

Proceduralne drzewo z kartkami listowia (alpha-clip billboards) wygladalo raz jak
zwarta korona, a raz jak kilka osobnych kul zawieszonych na golym patyku - przy
identycznej liczbie kartek i identycznym koszcie w trojkatach. Podkrecanie liczby
kartek nic nie dawalo: korona nie robila sie gestsza, tylko drozsza.

## Przyczyna

Kartki nie siadaja gdziekolwiek - siadaja na PUNKTACH ZACZEPIENIA przygotowanych
przez szkielet (galezie). Nadmiar rozsuwa sie wzdluz nosnika, ale ma na to najwyzej
jeden odstep miedzy sasiednimi punktami, czyli kilka centymetrow. Gdy punktow jest
malo, kartki pietrza sie w jednym miejscu.

Zmierzone na tym samym drzewie w dwoch konfiguracjach:

| rozgalezienie | kartek na punkt | odstep sasiednich kepek | jak wyglada |
|---|---|---|---|
| geste | 2,1 | 38 mm | korona ciagla |
| rzadkie | 20,3 | 6 mm | osobne kule, gole drewno miedzy |

Kartka miala 30 cm dlugosci. Przy odstepie 6 mm kartki po prostu leza jedna na
drugiej - placi sie trojkatami za zielen, ktorej nie widac, a dodatkowo rosnie
overdraw, bo alpha-clip nakladaja sie warstwami.

## Regula

**Stosunek `liczba_instancji / liczba_punktow_zaczepienia` to najwazniejsza liczba
calego systemu rozsypywania.** Trzymaj ja ponizej okolo 3.

Gdy jest za wysoka, lekarstwem jest **wiecej nosnikow** (gestszy szkielet), nigdy
wiecej instancji. Wiecej instancji na tej samej liczbie punktow to czysta strata.

Odwrotnie tez: gdy stosunek spada ponizej 1, jest zapas miejsca i mozna **powiekszyc
same instancje** - rozmiar kartki nie kosztuje ani jednego trojkata.

## Jak to wykryc u siebie

Wystaw ten stosunek jako pole diagnostyczne obok liczby trojkatow. Do tej pory
liczylismy trojkaty, wysokosc i szczelnosc, a ta jedna liczba - ktora decydowala
o wygladzie - nie byla nigdzie widoczna. Kontrola automatyczna powinna miec dowod
w GEOMETRII (mediana odleglosci do najblizszej sasiedniej instancji), a nie tylko
dzielic dwie wlasne liczby przez siebie.

## Przenosnosc

Dotyczy kazdego rozsypywania instancji po szkielecie albo powierzchni: listowie,
trawa, gruz, naloty, dekale, tlum NPC na sciezkach. Wszedzie tam, gdzie "wiecej
instancji" wydaje sie oczywistym lekarstwem na rzadki wyglad, a naprawde trzeba
dolozyc miejsc, na ktorych moga usiasc.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260728-1320-mniej-wiekszych-kartek|Polowa kartek o rozmiarze wiekszym o jedna trzecia wyglada tak samo]] - wspolne: alpha-cards, listowie, optymalizacja
- [[20260728-1140-miernik-ktory-klamie-inaczej|Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac]] - wspolne: proceduralne-generowanie, pomiar
- [[20260728-1900-miara-z-degeneracja|Miara optymalizowana samotnie znajduje rozwiazanie zdegenerowane]] - wspolne: optymalizacja, pomiar
- [[20260727-1921-kamera-z-siatki-podlogi-nie-z-sylwetki|Kamerę odtwarzaj z regularnej struktury sceny, nie z sylwetki modelu]] - wspolne: pomiar, blender
- [[20260727-2320-sylwetka-nie-rozdziela-czesci-ktore-sie-stykaja|Sylwetka nie rozdziela dwóch rzeczy, które się stykają - i milczy o tym]] - wspolne: pomiar, blender
- [[20260727-0525-jeden-suwak-dwie-role|Jeden suwak sterujacy dwiema roznymi rzeczami]] - wspolne: proceduralne-generowanie, blender
<!-- /POWIAZANE:auto -->
