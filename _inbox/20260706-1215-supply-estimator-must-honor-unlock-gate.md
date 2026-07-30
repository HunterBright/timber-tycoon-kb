---
title: Estymator podaży/dostepnosci MUSI respektowac te sama bramke odblokowan co produkcja
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-06'
project: Kerf - Sawmill Tycoon
tags:
- economy
- unlock-gating
- reputation
- demand-supply
- anti-softlock
- order-generation
applies_to: []
source: ''
suggested-category: gameplay-systems/lessons
---

# Estymator podaży/dostepnosci MUSI respektowac te sama bramke odblokowan co produkcja

## Kontekst
Gra tycoon: gracz odblokowuje lepsze zasoby wraz z postepem (u nas: lepsze gatunki drzew ->
lepszy opal, zablokowane strefami reputacji). Osobny system generuje popyt (zamowienia NPC) i,
zeby uniknac softlocka, wazy/capuje zamowienia wg "ile gracz jest w stanie ZDOBYC" (estymator
podazy).

## Objaw
NPC zamawiali towar wyzszego tieru (dobry/premium opal) ZANIM gracz mogl go wyprodukowac.
Wygladalo to jak brak bramki reputacji na zamowieniach.

## Prawdziwa przyczyna
Produkcja byla poprawnie zablokowana (drzewa lepszych gatunkow w strefach odblokowywanych
reputacja). ALE estymator podazy liczyl KAZDY zasob wg jego stanu fizycznego (drzewo "stoi" ->
policz jego przyszle klody), NIE sprawdzajac, czy jest juz ODBLOKOWANY. Zablokowane-ale-stojace
drzewa zawyzaly "osiagalny" wyzszy tier > 0, wiec generator popytu uznawal go za zamawialny.
Krotko: DEMAND-side wyprzedzil SUPPLY-side, bo estymator ignorowal bramke, ktora respektowala
produkcja.

## Lekcja (transferowalna)
Kazdy kod liczacy "co gracz moze zdobyc/zrobic" pod zamowienia, questy, podpowiedzi czy AI popytu
musi sprawdzac TE SAME bramki odblokowan (poziom/strefa/reputacja/zakup), ktorych pilnuje sciezka
produkcji. Stan fizyczny obiektu (istnieje/gotowy) to nie to samo co "dostepny dla gracza teraz".

## Fix (wzorzec)
Wystawic z bramki produkcji publiczny, tani odczyt stanu ("czy odblokowane") i wpiac go jako
guard w estymatorze podazy - pomijac zablokowane zasoby. Bramka pozostaje JEDNYM zrodlem prawdy
(u nas: dane strefy TreeZone). Dzieki temu popyt sam podaza za odblokowaniami, bez sztywnych
progow (magic-numbers) duplikujacych dane bramki.

## Anty-wzorzec (odrzucony)
Wpisanie w generator popytu sztywnych progow "tier X od poziomu N". Dziala, ale duplikuje dane
odblokowan w drugim miejscu - przy zmianie przypisania zasobu do poziomu trzeba pamietac o obu.

## Sygnaly ze to Twoj przypadek
- Osobny "anty-softlock"/estymator liczy podaz przez FindObjectsByType i sprawdza tylko stan obiektu.
- Bramka odblokowan zyje na innym obiekcie (parent/manager) niz liczony zasob.
- Popyt/podpowiedzi pojawiaja sie dla tresci, ktorej gracz nie moze jeszcze uzyc.
