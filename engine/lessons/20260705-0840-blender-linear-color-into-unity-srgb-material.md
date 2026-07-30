---
title: Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-05'
project: Kerf - Sawmill Tycoon
tags:
- blender
- unity
- color-space
- linear
- srgb
- material
- gamma
- shader
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)

## Objaw
Element (liście, tkanina, cokolwiek), którego kolor pochodzi z material Color property (nie z tekstury),
wygląda o WIELE ciemniej w Unity niż to co zaakceptowano we viewport Blendera. Oryginalne assety z BIAŁYM
tintem (kolor niesiony przez vertex color / teksturę) wyglądają dobrze; problem tylko tam, gdzie kolor jest
wpisany jako liczby w Color property materiału.

## Przyczyna
Blender przechowuje Principled Base Color w przestrzeni LINIOWEJ, ale viewport pokazuje go przez transform
sRGB - więc autor WIDZI jasną wartość i ją akceptuje. Te same liczby (np. 0.10, 0.22, 0.08) wpisane wprost
do Unity material Color property: w projekcie Linear Unity traktuje Color property jako sRGB/gamma i robi
sRGB->linear przy uploadzie na GPU. Efekt: shader dostaje ~2.4 potęgi CIEMNIEJSZY kolor (np. ~0.010,0.040,0.007)
- prawie czarny. To ta sama pułapka co "bake wychodzi za ciemny", tylko na kolorze materiału zamiast tekstury.

Klucz: material Color property w Unity Linear = interpretowany jako sRGB. Vertex colors i wartości z tekstur
sRGB przechodzą inną ścieżką, więc nie łapią tego podwójnego przeliczenia.

## Rozwiązanie
Wpisując do Unity kolor pochodzący z LINIOWEGO base color Blendera, przekonwertuj go gamma:
```csharp
mat.SetColor("_MyTint", blenderLinearColor.gamma);  // linear -> sRGB, Unity sRGB->linear cofnie to z powrotem
```
Wtedy shader dostaje z powrotem zamierzoną wartość liniową (tę, którą autor widział w Blenderze).
Alternatywnie: policz gamma-korektę raz i wpisz wartości sRGB wprost do .mat.

Uwaga: to NIE jest sprawa flagi sRGBTexture na .png (jeśli w ogóle jest tekstura) - to sprawa Color PROPERTY.
Sprawdź m_ActiveColorSpace w ProjectSettings (1 = Linear).

## Koszt gdy pominięte
Wygląda jak zły model / złe oświetlenie / zła tekstura - łatwo szukać w złym miejscu. Diagnostyka: jeśli
identyczny shader z BIAŁYM tintem wygląda dobrze, a z kolorowym jest za ciemno, to prawie na pewno to.
