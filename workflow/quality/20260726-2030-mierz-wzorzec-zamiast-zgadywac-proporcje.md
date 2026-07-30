---
title: Gdy rezyser mowi "wyglada zle", zmierz cos, co juz zaakceptowal
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-26'
project: Kerf - Sawmill Tycoon
tags:
- proporcje
- kalibracja
- pomiar
- modele-3d
- feedback
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Gdy rezyser mowi "wyglada zle", zmierz cos, co juz zaakceptowal

## Problem

Ocena wizualna od rezysera brzmi: "tulow wyglada nienaturalnie", "glowa wyglada
bardzo zle", "dlon rozszerza sie zbyt mocno". To sa prawdziwe i trafne uwagi, ale
nie da sie ich bezposrednio zamienic na liczby. Kolejne rundy poprawek "na oko"
trafiaja raz lepiej, raz gorzej, i latwo zamienic jedna wade na druga.

## Wzorzec

W projekcie prawie zawsze istnieje juz cos, co ten sam czlowiek ZAAKCEPTOWAL:
starszy model, asset z paczki, postac z konkurencyjnej gry, cokolwiek. To jest
miara. Zamiast dyskutowac o guscie, **potnij wzorzec i swoja prace tym samym
kodem i porownaj liczby**.

Praktycznie:

1. Zaimportuj wzorzec i wyznacz jego wysokosc.
2. Tnij plastrami co 1% wysokosci, dla kazdego plastra policz szerokosc
   i glebokosc konturu zawierajacego os ciala.
3. Zrob dokladnie to samo na swojej siatce.
4. Zestaw w tabeli.

W tym przypadku jedno takie zestawienie wyjasnilo naraz TRZY osobne uwagi:

| miejsce | wzorzec | nasza | wniosek |
|---|---|---|---|
| barki | 0,354 m | 0,443 m | 25% za szeroko -> "tulow nienaturalny" |
| klatka, glebokosc | 0,263 | 0,236 | za plytko -> plaski stozek zamiast beczki |
| glowa, linia oczu | 0,189 | 0,170 | za waska -> "orzeszek na patyku" |
| grubosc dloni | 0,079 | 0,032 | za cienka -> stad wachlarz od nadgarstka |

Uwaga o dloni jest tu najciekawsza: rezyser powiedzial "rozszerza sie zbyt mocno",
czyli opisal ZBYT DUZA szerokosc. Pomiar pokazal, ze prawdziwym problemem byla
ZBYT MALA grubosc - dlon byla plaska, wiec zeby miec objetosc, musiala byc szeroka.
Bez pomiaru poprawka poszlaby w zla strone.

## Pulapka: sprawdz, czy wzorzec jest tym samym rodzajem obiektu

Nasz wzorzec to postac UBRANA. Ma biodra 0,347 przy tulowiu 0,375, czyli
praktycznie zadnej litery V - bo to kurtka, nie cialo. Kalibracja 1:1 zabrala
nagiej postaci bazowej sylwetke i sprawila, ze czytala sie kobieco.

Wniosek: przenoś z wzorca te wymiary, ktore opisuja TO SAMO (dlugosc stopy,
szerokosc glowy, grubosc dloni), a przy tych, gdzie ubranie zmienia obraz
(obwod tulowia, talia), traktuj wzorzec jako gorna granice, nie jako cel.

## Kiedy uzywac

- gdy dostajesz ocene wizualna, ktorej nie umiesz przelozyc na liczbe
- gdy kolejne rundy poprawek zaczynaja chodzic w kolko
- ZAWSZE przed strojeniem proporcji czegokolwiek, co ma pasowac do istniejacej
  reszty gry

## Koszt

Jeden skrypt tnacy, kilkanascie minut. Zwrocil sie po pierwszym uruchomieniu.

## Pulapka techniczna warta zapamietania

Automatyczne wykrywanie osi pionowej po "najwiekszym rozmiarze bryly" ZAWODZI
dla postaci w pozie T: rozpietosc ramion rowna sie wzrostowi i czesto go
przekracza. Trzeba wymusic os albo wykrywac ja inaczej.
