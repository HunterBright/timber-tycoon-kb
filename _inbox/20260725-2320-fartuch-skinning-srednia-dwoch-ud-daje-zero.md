---
title: Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- rigging
- skinning
- cloth
- apron
- skirt
- character
- blender
- unity
- walk-cycle
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
---

# Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie

## Zadanie

Sprzedawca ma fartuch. Wymóg: fartuch ma poruszać się z nogami przy chodzie, a nogi
nie mogą przez niego przebijać. Bez symulacji tkaniny, samym ważeniem kości.

## Pierwszy pomysł i dlaczego jest zły

Intuicyjne rozwiązanie: dół fartucha ważymy **50/50 na oba uda**, żeby jechał
do średniej ich ruchu, a nie rozrywał się na dwie połowy przy nożycach.

To rozumowanie ma dziurę: **w chodzie ruch nóg jest ANTYSYMETRYCZNY**. Jedna noga idzie
w przód o +25 stopni, druga w tył o -25 stopni. Średnia wynosi **dokładnie zero**.
Fartuch stoi nieruchomo, podczas gdy kolano jedzie 176 mm do przodu.
Zmierzone przebicie: **127 mm** przy 25 stopniach, 184 mm przy 35.

Żaden statyczny prześwit tego nie uratuje. Fartuch stał 41 mm przed udem i to nie miało
znaczenia, bo kolano przejeżdża cztery razy tyle.

## Rozwiązanie, które działa

**Podzielić dół fartucha WZDŁUŻ OSI PIONOWEJ i każdą połowę przypiąć do SWOJEGO uda**,
z płynnym przejściem przez środek (u nas przejście na szerokości 110 mm).

- góra fartucha: klatka i tułów
- środek: biodra
- dół: waga uda rośnie ku dołowi (u nas do 0,85), rozdzielona lewo/prawo według X

Każda połowa jedzie za swoją nogą, więc przebicia nie ma. Środek się rozciąga jak
tkanina, co przy płaskich kolorach i sylwetce postaci w grze jest niewidoczne.

Wynik po zmianie: prześwit **14 mm w spoczynku i ROSNĄCY do 107 mm przy wymachu**,
zamiast malejącego do minus 127 mm.

## Pułapka po drodze: nazwa kości a strona świata

Kość nazwana `.L` wcale nie musi stać po tej stronie, po której się wydaje. U nas lustro
tabeli kości postawiło `UpperLeg.L` po **dodatniej** stronie X. Przypisanie ujemnej połowy
fartucha do `.L` sprawiło, że połówki jechały za **przeciwnymi** nogami i przebicie urosło
ze 127 mm na **259 mm**. Sprawdzać pozycję kości w świecie, nie jej nazwę.

## Pułapka ważniejsza: pomiar porównywał różne wysokości

Pierwsza wersja testu brała **najbardziej cofnięty punkt fartucha z całego zakresu**
i **najbardziej wysunięte kolano z całego zakresu**, czyli fartuch z góry kontra noga
z dołu. Taki pomiar zawsze wychodził ujemny i **nie zależał od wag**.

Zdemaskowały go dwie różne wersje ważenia dające **wynik identyczny co do dziesiątej
milimetra**. To jest sygnał alarmowy: jeśli zmiana, która powinna wpłynąć na wynik,
nie zmienia go ani o włos, to nie mierzysz tego, co myślisz.

Poprawny pomiar: podzielić wysokość na pasy (u nas 16) i w KAŻDYM pasie osobno
porównać tył fartucha z przodem nogi, po każdej stronie osobno.

## Reguła

Test przebicia po trzech poprawkach dawał zielone światło i **właśnie dlatego nie wolno
mu było ufać**. Dopisany tryb dowodowy: ta sama procedura na fartuchu ważonym TYLKO
na biodrach musi zgłosić przebicie. Wynik: z wagami +14 mm, bez nich -88 mm,
różnica 102 mm. Dopiero to dowodzi, że test mierzy ważenie, a nie kształt.

Patrz też: [[feedback_probe_must_be_able_to_fail]].
