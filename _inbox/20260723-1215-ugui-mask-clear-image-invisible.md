---
title: 'uGUI: Mask z obrazkiem Color.clear = NIEWIDZIALNA zawartosc (a transformy klamia, ze wszystko gra)'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-23'
project: Kerf - Sawmill Tycoon
tags:
- unity
- ugui
- mask
- rectmask2d
- dropdown
- scrollrect
- cullTransparentMesh
- ui-w-kodzie
applies_to: []
source: ''
suggested-category: engine/lessons
---

# uGUI: Mask z obrazkiem Color.clear = NIEWIDZIALNA zawartosc (a transformy klamia, ze wszystko gra)

## Objaw
Rozwiniete listy (Dropdown) i przewijane obszary (ScrollRect) budowane w kodzie pokazuja
puste tlo - zawartosc (pozycje listy, teksty) nie renderuje sie W OGOLE, mimo ze:
- opcje sa w danych (options.Count poprawne),
- sklonowane pozycje istnieja, sa aktywne i maja POPRAWNE pozycje/rozmiary,
- warstwy sortowania sa dobre.
Gracz widzi "liste z jedna pozycja" (naglowek zamknietej kontrolki) albo puste okno.

## Mechanizm
Przepis-pulapka przy budowie viewportu w kodzie:
```csharp
viewport.AddComponent<Image>().color = Color.clear;   // <- ZLO
viewport.AddComponent<Mask>().showMaskGraphic = false;
```
Mask zapisuje szablon (stencil) rysujac SWOJ obrazek. CanvasRenderer ma domyslnie
wlaczone cullTransparentMesh - obrazek o alpha 0 jest WYCINANY z renderowania, wiec
szablon nigdy nie powstaje, a wszystkie dzieci maski odpadaja na tescie szablonu.
showMaskGraphic=false NIE jest problemem (ukrywa kolor przez colorMask, obrazek dalej
rysuje szablon) - problemem jest alpha 0 KOLORU obrazka.

## Naprawa
Do prostokatnych viewportow: **RectMask2D** zamiast pary Image+Mask - zero obrazka,
zero szablonu, przycinanie po prostokacie:
```csharp
viewport.AddComponent<RectMask2D>();
```
Mask zostaje tylko tam, gdzie przyciecie ma miec KSZTALT obrazka (np. pasek w ksztalcie
deski) - wtedy obrazek musi miec alpha > 0 (showMaskGraphic=false moze zostac).
Obejscie widywane w kodzie: alpha = 0.01 zamiast 0 - dziala, ale to plaster, nie naprawa.

## Lekcja o testowaniu (wazniejsza niz sama pulapka)
Check "strukturalny" (liczba pozycji, pozycje, rozmiary, activeInHierarchy) PRZECHODZI
na zepsutym buildzie - transformy sa zdrowe, znika dopiero piksel. Automat, ktory ma to
lapac, musi porownac WYRENDEROWANY OBRAZ: zrzut ekranu + probka piksela ze srodka
pierwszej pozycji vs kolor tla szablonu (rozne = widoczne). Uwaga na fade-in listy
Dropdowna (0.15 s, AlphaFadeList) - zrzut po 1-2 klatkach lapie liste przy ~kilku %
przezroczystosci i falszywie melduje pustke.

## Sygnaly, ze to TEN bug
- tlo listy/panelu widac (rodzic poza maska), zawartosci nie,
- w hierarchii wszystko aktywne i na miejscu,
- gdzies w projekcie ktos juz "naprawil" podobne okno przez alpha=0.01.
