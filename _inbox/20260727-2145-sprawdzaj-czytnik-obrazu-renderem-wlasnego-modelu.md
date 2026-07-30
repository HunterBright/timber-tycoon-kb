---
title: Czytnik obrazu sprawdzaj renderem własnego modelu, nie obrazkiem, który sam sobie narysowałeś
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-27'
project: Kerf - Sawmill Tycoon
tags:
- testy
- pomiar
- obraz
- render
- blender
- kontrola
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Czytnik obrazu sprawdzaj renderem własnego modelu, nie obrazkiem, który sam sobie narysowałeś

## Problem

Kod, który coś mierzy ze zdjęcia (obrys, krawędzie, kolor, położenie), zwykle
dostaje samotest na obrazkach generowanych przez ten sam kod. Taki test łapie
regresje, ale nie odpowiada na pytanie, które naprawdę boli: **ile ten pomiar
myli się na PRAWDZIWYM obrazku?** Syntetyczny obrazek nie ma cieniowania,
antyaliasingu, perspektywy ani zmiennej grubości kreski.

## Wzorzec

Jeśli mierzysz coś, co sam potrafisz WYPRODUKOWAĆ, zrób trzeci przebieg:
wyrenderuj własny model — taki, w którym znasz prawdę co do milimetra —
w stylu jak najbliższym referencji i puść na niego ten sam czytnik.

Trzy przebiegi, każdy odpowiada na inne pytanie:

1. **obrazek syntetyczny** — czy kod nie ma regresji (szybkie, w samoteście),
2. **własny render, kamera rzutująca** — jaki jest CZYSTY błąd czytnika,
3. **własny render, kamera zmierzona z referencji** — ile dokłada kamera.

Różnica między 2 a 3 to pomiar wkładu perspektywy. U nas: 1,1 mm wobec
30,6 mm, co od razu pokazało, że problemem nie jest czytnik, tylko sposób
przeliczania pikseli na wysokość.

## Pułapka: render musi być tak trudny jak referencja, nie łatwiejszy

Pierwsza wersja testu była PRZYPADKIEM trudniejsza od rzeczywistości i przez
to bezwartościowa:

- kreski miały zwykły ciemny materiał, więc światło rozjaśniało je do połowy
  tonu (kontrast 30 wobec 123 w referencji) — potrzebna była emisja o mocy
  zero, czyli powierzchnia nieczuła na światło,
- rurki kresek leżały dokładnie w płaszczyźnie ścianek i biły się o te same
  piksele, przez co kreska wychodziła przerywana — trzeba je wypchnąć
  na zewnątrz powierzchni,
- maska modelu brana z progu jasności ucinała zacieniony dół ciała, więc
  czytnik dostawał dziurawe wnętrze; maska musi iść z kanału alfa.

Zanim ogłosisz wynik testu, porównaj LICZBOWO jedną cechę obrazu testowego
z referencją (u nas: jasność kreski wobec jasności ścianki obok). Jeśli się
rozjeżdżają, test mierzy własny render, a nie czytnik.

## Kiedy to się nie uda

Wzorzec działa tylko dla wielkości, które umiesz sam wytworzyć. Jeśli
referencja pokazuje coś, czego nie potrafisz zbudować (u nas: twarz), nie
zrobisz sobie prawdy — i wtedy trzeba mierzyć zgodność między niezależnymi
ujęciami zamiast z prawdą.
