---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [3d-generation, trellis, image-to-3d, asset-pipeline, reference-art]
severity: medium
time_lost: "20 min recon (uniknieto ~1 h generowania widokow na marne)"
date: 2026-07-26
status: draft
applies_to: [TRELLIS.2, image-to-3D]
---

# Generator obraz-do-3D karmi sie JEDNYM obrazkiem, a nie kompletem widokow

## Problem
Zamowienie brzmialo: "wygeneruj duzo referencji postaci z kazdej strony, zeby generator
jednoznacznie widzial kazdy element ubioru i twarzy, i pusc je przez TRELLIS". Zalozenie
jest naturalne (wiecej widokow = lepszy model), ale w TRELLIS.2 jest po prostu niewykonalne.

## Root cause
`Trellis2ImageTo3DPipeline.run(image, ...)` przyjmuje POJEDYNCZY obraz. Wewnetrznie robi
`get_cond([image], 512)`, a otrzymany warunek jest rozgłaszany na partie probek
(`noise = torch.randn(num_samples, ...)`). Podanie listy N obrazkow nie skleja widokow
w jeden model - albo rozjezdza sie rozmiar partii, albo powstaje N osobnych modeli.
Trybu wielowidokowego, ktory byl w pierwszym TRELLIS (`run_multi_image` z wariantami
"stochastic"/"multidiffusion"), w TRELLIS.2 nie ma ani w kodzie, ani w opisie autorow.

Sprawdzenie zajmuje 2 minuty: `grep -rn "def run" trellis2/pipelines/*.py` i przeczytanie
sygnatury, plus `grep -rn "multi" trellis2/`.

## Solution
Komplet widokow nadal jest potrzebny, tylko do INNYCH trzech rzeczy niz warunkowanie:
1. **Wybor wejscia** - generator odpalany kilka razy z roznych pojedynczych widokow
   (przod, 3/4 z lewej, 3/4 z prawej); z kilku bryl wybiera sie najlepsza. Na RTX 4090
   jeden przebieg to minuty, wiec selekcja jest tania.
2. **Mapa elementow do malowania** - zeby wiedziec, jakiego koloru jest kamizelka z tylu
   i gdzie konczy sie pas odblaskowy, trzeba widziec postac dookola.
3. **Kontrola wyniku** - porownanie gotowej bryly z zamiarem z 8 stron.

Do warunkowania wybiera sie JEDEN widok 3/4 (nie plaski przod): modele obraz-do-3D
najlepiej rozstrzygaja z niego glebokosc.

## What didn't work
- Sklejanie widokow w jeden arkusz ("contact sheet") i podanie go jako jednego obrazka:
  generator traktuje to jak scene z wieloma obiektami albo jak plaska plansze.
- Zakladanie, ze skoro poprzednia wersja narzedzia miala tryb wielowidokowy, to nowa tez ma.

## Transferability
Dotyczy calej rodziny generatorow obraz-do-3D (TRELLIS, TripoSG, SPAR3D, Hunyuan3D):
warunkowanie jest na pojedynczym obrazie, a "wiecej widokow" jest strategia SELEKCJI
i KONTROLI, nie strategia wejscia. Wplywa na planowanie czasu: nie warto generowac
kilkunastu widokow "dla generatora", warto generowac je dla siebie.

## Related
- [[20260725-1830-image-model-cannot-force-figure-proportions]]
