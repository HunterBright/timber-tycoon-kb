---
title: Generator obrazkow nie da sie zmusic opisem do proporcji figury (7,5 glowy)
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- ai-image
- comfyui
- qwen-image
- character-reference
- proportions
- prompt-engineering
applies_to: []
source: ''
suggested-category: pipeline/anti-patterns
---

# Generator obrazkow nie da sie zmusic opisem do proporcji figury (7,5 glowy)

## Problem

Potrzebna byla referencja postaci o **naturalnych ludzkich proporcjach: 7,5 glowy na wzrost**,
w kreskowkowym stylu 3D. Wszystkie modele obrazkowe maja silny nawyk: "kreskowkowa postac 3D"
= duza glowa, krotkie nogi. Poprzednia referencja miala glowe na 31% wzrostu (3,2 glowy)
i model z niej zrobiony zostal odrzucony.

## Co probowane i ZMIERZONE (Qwen-Image + Qwen-Image-Edit 2509, fp8, RTX 4090)

| podejscie | wynik (glow na wzrost) |
|---|---|
| opis z liczbami ("head fits 7.5 times", "legs half the height") | 3,0-4,4 |
| to samo, ale BEZ slow "toy figure"/"figurine", z "small head, long legs" i tymi slowami w opisie negatywnym | 4,9-6,8 |
| plaski szablon proporcji przerobiony na styl (denoise 0,75 i 1,0) | 7,17 — ale wynik zostal **plaski**, bez bryl |
| skladanie dwoch obrazkow (postac + szablon proporcji) w Edit 2509 | 5,1 |
| rozciagniecie gotowej postaci przez Edit ("head smaller, legs longer") | **6,87 — najlepsze** |
| drugie rozciagniecie na wyniku pierwszego | 6,72 — **gorzej**, model wraca do nawyku |

## Wniosek

1. **Slowa "toy figure" i "figurine" same z siebie skracaja figure.** Wyrzucenie ich z opisu
   pozytywnego i wpisanie do negatywnego (razem z "chibi, funko pop, bobblehead, big head,
   short legs") dalo najwieksza pojedyncza poprawe: z ~3 do ~5-6 glow.
2. **Sufit istnieje.** Ten model zbiega do ~6,8-6,9 glowy i dalej nie idzie. Iterowanie tego
   samego zabiegu **cofa** wynik, nie poprawia.
3. **Nie da sie miec jednoczesnie narzuconych proporcji i pelnego stylu.** Przy silnym trzymaniu
   szablonu wynik zostaje plaski; przy silnym stylu wracaja krepe proporcje. To nie kwestia
   doboru sily zaszumienia — to dwa konce tej samej suwaki.

## Co robic zamiast

Jesli proporcje MUSZA byc dokladne, obrazek nie jest zrodlem prawdy o proporcjach.
Referencje traktowac jako zrodlo **stylu i detalu** (twarz, kolory, kroj ubrania),
a proporcje wpisac **liczbami do skryptu modelujacego** i tam ich pilnowac kontrolami.
Rozjazd miedzy referencja i modelem trzeba wtedy zglosic decydentowi, nie zamiatac.

Dodatkowo: bok i tyl generowac **z jednego zatwierdzonego przodu** przez model edytujacy,
nie od zera — zmierzona zgodnosc odcieni miedzy widokami wyszla wtedy 8/9 elementow
w granicach 12/255. Generowanie kazdego widoku osobno bylo w poprzednim podejsciu
glownym zrodlem rozjechanych kolorow tego samego ubrania.
