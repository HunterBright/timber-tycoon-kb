---
type: pattern
project: Timber Tycoon
suggested-category: gamedesign/patterns
tags: [economy, orders, weighted-random, self-regulation, tycoon]
date: 2026-07-10
status: draft
---

# Losowanie zamówień ważone podażą wymaga PODŁOGI wag

## Problem
W tycoonie zamówienia NPC losowano proporcjonalnie do "osiągalnej podaży" produktu
(magazyn + co da się wyprodukować z dostępnego surowca). System samoregulujący i
anty-softlockowy (nie zamawia rzeczy nie do zdobycia), ALE ma patologię: produkt,
który dobrze się sprzedaje, ZNIKA z rynku - sprzedaż zbija zapas, zapas zbija wagę,
waga zbija sprzedaż (pętla dodatnia w złą stronę). Odpady/półprodukty, których nikt
nie zbiera (pniaki -> zrębki), pompują swoją wagę do 90%+. Objaw u playtestera:
"na L2 sprzedaję więcej zrębków niż opału, pellet nie schodzi w ogóle".

## Wzorzec
Zostaw wagę = podaż (samoregulacja + anty-softlock), ale dodaj PODŁOGĘ:
każdy produkt OSIĄGALNY (>= 1 szt.) dostaje wagę co najmniej
`floorFraction x NAJWIĘKSZA surowa waga` (u nas floorFraction = 0.15, wartość w SO).

- Frakcja MAKSIMUM zamiast stałej: skaluje się z liczbami (opał 200 vs zrębki 4:
  bez podłogi 98%/2%, z podłogą ~87%/13% - dalej premiuje nadwyżki, ale nic nie znika).
- Podłoga TYLKO dla osiągalnych (waga 0 zostaje 0) - anty-softlock nietknięty.
- Podłoga PRZED mnożnikami eventowymi (u nas "materiał dnia" x3), żeby boost działał
  też na podłogowanych.
- Cap ilości per zamówienie (min(maxAmount, osiągalne)) zostaje bez zmian.

## Bonus: audyt puli
Przy okazji wyszło, że jeden produkt (nawóz) NIGDY nie był zamawialny - brakowało go
w puli i w mapie produkt->maszyna, a bramka "produkt spoza mapy = niezamawialny" jest
cicha. Lekcja: przy dodawaniu produktu do gry sprawdź CAŁY łańcuch sprzedaży
(pula zamówień + bramka + estymator podaży), nie tylko produkcję. Warto mieć debug
menu drukujące tabelę % szans przy bieżącym stanie gry (u nas: Print Order Weights).

## Kiedy stosować
Każda gra z zamówieniami/klientami ważonymi stanem świata (tycoon, shopkeeper).
