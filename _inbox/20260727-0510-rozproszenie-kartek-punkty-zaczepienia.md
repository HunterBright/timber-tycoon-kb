---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [proceduralne-generowanie, blender, alpha-cards, listowie, optymalizacja, pomiar]
date: 2026-07-27
status: draft
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
