---
title: Wspolny kontrakt liczbowy dla kilku agentow, ktory nie nazywa osi
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- multi-agent
- blender
- asset-pipeline
- kontrakt
- low-poly
- szwy
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Wspolny kontrakt liczbowy dla kilku agentow, ktory nie nazywa osi

## Pulapka
Kilku agentow buduje rownolegle rozne czesci jednego modelu z jednego pliku z wymiarami
(tu: `kerf_contract.py`). Kontrakt podaje przekroje jako pary liczb w stylu
`(polowa_szerokosci, polowa_glebokosci)` i wyglada na jednoznaczny. Dla brył stojacych
w pionie faktycznie jest. Dla konczyny lezacej poziomo (ramie w pozie T, os X)
para liczb NIE MOWI, ktora idzie na Y, a ktora na Z.

## Dlaczego zawodzi
Kazdy agent rozstrzyga sam i kazdy rozstrzyga rozsadnie. Czesc bierze kolejnosc
cykliczna (X -> Y -> Z, wiec 1. liczba na Y), czesc czyta "szerokosc" jako pion.
Roznica jest mala (3 mm na promieniu), wiec zadna kontrola per-czesc jej nie zlapie:
kazdy raport jest zielony, kazda czesc pasuje do kontraktu. Rozjazd wychodzi dopiero
przy skladaniu, jako przekrecony o 90 stopni przekroj na szwie.

## Objawy
- wszystkie czesci maja "0 mm odchylki od K.SEAMS", a mimo to skora wychodzi przez rekaw
- rozjazd jest maly i tylko przy jednym szwie, wiec wyglada jak blad zaokraglenia
- 2 z 3 agentow maja tak samo (wieksze prawdopodobienstwo, ze poprawia sie mniejszosc)

## Poprawne podejscie
1. W kontrakcie nazywac osie wprost, nie polegac na slowie "szerokosc":
   zamiast `(0.029, 0.032)` dawac `{"y": 0.029, "z": 0.032}` albo funkcje
   `ring_for_seam(name)` zwracajaca gotowe punkty 3D. Punkt szwu ma byc DANY,
   nie odtwarzany przez kazdego agenta osobno.
2. Kazda czesc dopisuje do raportu PELNE wspolrzedne swojego pierscienia szwu.
   Wtedy porownanie dwoch raportow to 10 sekund, a nie debugowanie zlozonego modelu.
3. Kontrola ma mierzyc SKUTEK dla sasiada (ile mm skory wystaje poza rekaw przy
   obu interpretacjach), a nie tylko zgodnosc z wlasnym zalozeniem. Kontrola,
   ktora sprawdza wlasne zalozenie wzgledem samego siebie, zawsze przechodzi.
