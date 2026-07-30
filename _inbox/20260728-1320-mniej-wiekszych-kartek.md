---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [optymalizacja, alpha-cards, listowie, pomiar, overdraw, proceduralne-generowanie]
date: 2026-07-28
status: draft
---

# Polowa kartek o rozmiarze wiekszym o jedna trzecia wyglada tak samo

## Pytanie

Czy da sie odchudzic drzewo z kartek alpha-clip (listowie), zeby wygladalo
identycznie? Odpowiedz "na oko" byla niepewna, wiec zmierzylem.

## Jak zmierzyc "wyglada tak samo"

Renderowanie z PRZEZROCZYSTYM TLEM i liczenie pikseli, ktore maja alfe powyzej
0,5 - z trzech katow (0, 45, 90 stopni). Ta jedna liczba lapie naraz zarys korony
i dziury w srodku: kartek moze ubyc, ale jesli sylwetka i wypelnienie zostaja te
same, liczba sie nie zmienia.

GOTCHA, ktora zjadla pierwsze podejscie: liczenie pikseli po KOLORZE TLA nie
dziala, bo obrobka tonalna (AgX/Filmic) zmienia kolor tla w renderze. Pierwsza
wersja pomiaru zwracala identyczna liczbe dla kazdego ustawienia - byla to po
prostu liczba wszystkich pikseli kadru. Alfa jest odporna na obrobke tonalna.

## Wynik

Powierzchnia kartki rosnie z KWADRATEM jej rozmiaru, wiec zeby laczna powierzchnia
zostala ta sama przy polowie kartek, rozmiar wystarczy przemnozyc przez pierwiastek
z dwoch (okolo 1,41). Pomiar pokazal, ze w praktyce wystarcza MNIEJ - okolo 1,21 -
bo przy duzej liczbie kartek znaczna ich czesc lezy JEDNA NA DRUGIEJ i ta
powierzchnia jest zmarnowana.

Zmierzone na jednym drzewie (pokrycie w pikselach, wzorzec = 100%):

| kartek | rozmiar | trojkaty listowia | pokrycie |
|---|---|---|---|
| 2000 | 1,45 | 8000 | 100,0% |
| 1000 | 1,76 | 4000 | 99,9% |
| 700 | 1,97 | 2800 | 99,6% |
| 500 | 2,17 | 2000 | 100,0% |

Wzrokowo 2000 i 1000 sa nie do odroznienia. Przy 700 kepy zaczynaja czytac sie
jako osobne grudki, przy 500 w koronie widac dziury - mimo ze SUMA pikseli sie
zgadza. Czyli miernik pokrycia jest konieczny, ale niewystarczajacy: powyzej
pewnego rozmiaru kartki ten sam "metraz" zieleni uklada sie w grudki zamiast
w liscie. Granica lezy okolo polowy wyjsciowej liczby.

## Drugi zysk, wazniejszy od trojkatow

Polowa kartek to takze polowa warstw przezroczystosci nakladajacych sie na siebie
(overdraw). Przy alpha-clip to zwykle wiekszy koszt niz same trojkaty, bo kazda
warstwa to ponowne rysowanie tego samego piksela.

## Regula

Zanim dolozysz kartek, zmierz, ile ich lezy na sobie. Przy geometrii rozsypywanej
po szkielecie oplaca sie ciac liczbe i wyrownywac rozmiarem, dopoki pokrycie stoi -
zysk siega jednej trzeciej wszystkich trojkatow modelu przy zerowej roznicy
w wygladzie. Granice wyznacza moment, w ktorym pojedyncza kartka robi sie na tyle
duza, ze czyta sie jako grudka.
