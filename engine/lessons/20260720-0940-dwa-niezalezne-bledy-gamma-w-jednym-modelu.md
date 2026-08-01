---
title: '"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- gamma
- przestrzen-liniowa
- vertex-color
- materialpropertyblock
- blender-fbx
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# "Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw

## Objaw

Nowy model auta w grze wygląda na wyblakły: opony jasnoszare zamiast prawie czarnych,
szyby ledwo ciemniejsze od lakieru, kremowy lakier prawie biały. Renderowany w Blenderze
ten sam model wygląda **poprawnie**.

## Przyczyna nr 1: kolory w siatce eksportowane jako sRGB

Projekt renderuje w przestrzeni **liniowej** (`m_ActiveColorSpace: 1`). Shader używa koloru
z siatki wprost jako wartości liniowej:

```hlsl
float3 color = input.color.rgb * _LeafTint.rgb * lighting * _Brightness;
```

Blender przechowuje `BYTE_COLOR` jako bajty zakodowane w sRGB. Eksport FBX z
`colors_type='SRGB'` zapisuje właśnie te zakodowane wartości. Unity podaje je do shadera
bez konwersji, więc **kolor 0.11 dociera jako 0.37**.

Naprawa: `colors_type='LINEAR'` przy eksporcie. Zmierzone po naprawie: opona zadana jako
0.110 dociera jako **0.110**.

## Przyczyna nr 2: kolor wysyłany przez MaterialPropertyBlock

Niezależnie od powyższego, losowy lakier ustawiany był tak:

```csharp
block.SetColor("_BaseColor", new Color(r/255f, g/255f, b/255f));
```

`MaterialPropertyBlock.SetColor` **nie robi żadnej korekty gamma**. Wartość z zapisu
szesnastkowego jest w sRGB, więc w projekcie liniowym każdy kolor wychodzi za jasny.
Butelkowa zieleń `#3C5641` wyświetlała się jako wyblakła średnia zieleń.

Naprawa: `.linear` na kolorze przed zapisaniem go do puli.

## Dlaczego to się nie ujawniło wcześniej

Poprzedni model używał **tego samego kanału** (MaterialPropertyBlock, ta sama paleta),
ale jego materiał miał podpiętą **teksturę brudu**. Mnożenie przez teksturę przyciemniało
wynik i przypadkiem maskowało błąd. Nowy materiał tekstury nie ma, więc wada wyszła
na wierzch dopiero teraz.

> To jest ogólniejszy wzorzec: **usunięcie tekstury odsłania błędy gamma, które
> mnożenie przez nią maskowało.** Gdy upraszczasz materiał, sprawdź kolory od nowa.

## Jak to wykrywać, a nie zgadywać

Wzrokowo "za jasne" jest bezużyteczne, bo nie wiadomo, o ile i gdzie. Zamiast tego:

```csharp
float darkest = 1f;
foreach (var mf in fbx.GetComponentsInChildren<MeshFilter>(true))
    foreach (var c in mf.sharedMesh.colors)
        darkest = Mathf.Min(darkest, Mathf.Max(c.r, Mathf.Max(c.g, c.b)));
```

Porównaj z wartością, którą **zadałeś w modelu**. Zgadza się co do trzeciego miejsca
po przecinku - gamma jest dobra. Wyszło mniej więcej `srgb(zadana)` - masz błąd
o jeden krok gamma.

Ta jedna liczba zamienia spór o wygląd w fakt.

## Pułapka przy weryfikacji

Kontrola szukająca ścianek o **konkretnym kolorze** (np. reflektorów po wartości RGB)
przestaje działać przy każdej zmianie przestrzeni barw. Jeśli check ma pilnować
**kierunku** albo **położenia**, szukaj ścianek najjaśniejszych albo najciemniejszych
w danym materiale, a nie o zadanej wartości. Wtedy jest odporny na zmiany kolorystyki.

## Uwaga procesowa

Edycja pliku `.cs` i natychmiastowe uruchomienie Unity w trybie wsadowym to **wyścig**:
Unity potrafi wystartować z jeszcze starą skompilowaną wersją. Objaw: skrypt "zadziałał",
ale efekt jest taki jak przed zmianą. Sprawdzaj datę modyfikacji artefaktu, a nie sam
kod wyjścia.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[vertex-color-gamma-correction-blender-to-unity|Vertex Color Gamma Correction Blender → Unity]] - wspolne: gamma, vertex-color
- [[20260702-0651-mpb-does-not-toggle-keywords|Lekcja: MaterialPropertyBlock NIE włącza keywordów shadera (emisja niewidoczna)]] - wspolne: materialpropertyblock, urp
- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] - wspolne: vertex-color, urp
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] - wspolne: vertex-color, urp
- [[20260721-1845-tint-palette-on-textured-mesh-neutralize-average|Paleta kolorow na TEKSTUROWANEJ siatce: dziel tint przez sredni kolor tekstury]] - wspolne: materialpropertyblock, urp
<!-- /POWIAZANE:auto -->
