---
title: Edycja wygładzonego kształtu w rastrze — licz różnicę pokrycia, nie przemalowuj
type: pattern
status: draft
confidence: high
verified: ''
date: 2026-08-11
project: Discord_Studio (branding MGDB Studio)
tags:
- obrobka-obrazu
- antyaliasing
- logotyp
- numpy
- subpixel
applies_to: []
source: ''
suggested-category: workflow/patterns
---

# Edycja wygładzonego kształtu w rastrze — licz różnicę pokrycia, nie przemalowuj

## Kiedy

Trzeba zmienić kształt litery, ikony albo maski w gotowym pliku rastrowym — takim, który
ma wygładzone krawędzie i teksturę (gradient, ziarno), a nie ma warstw ani wektora.

## Wzorzec

Piksel na wygładzonej krawędzi to mieszanka: `piksel = litera·pokrycie + tło·(1 − pokrycie)`.
Edycja to zmiana **pokrycia**, nie zamalowanie pikseli.

```
pokrycie_stare = (piksel − tło) / (litera − tło)        # odczytane z OBRAZU
pokrycie_nowe  = analityczne, z nadpróbkowania nowego kształtu
wynik = piksel + (pokrycie_nowe − pokrycie_stare) · (litera − tło)
```

Tam, gdzie kształt się nie zmienia, różnica wynosi zero i piksel zostaje **bit w bit** —
z gradientem i ziarnem, których żadne odtwarzanie barwy nie trafia.

Trzy rzeczy, bez których to nie zadziała:

1. **Barwa litery jako model, nie próbka.** Litera ma gradient. Próbka z jednej kolumny
   dała przy przesunięciu o 30 px błąd 9 poziomów, czyli widoczny pionowy szew.
   Płaszczyzna dopasowana metodą najmniejszych kwadratów (`L = a·x + b·y + c`) miała
   błąd średnio 1,4 poziomu.
2. **Położenie krawędzi z pola pokrycia, nie z progu jasności.** Próg (jasność > 90)
   dawał x=583,46, a suma `(1 − pokrycie)` w oknie dała x=583,92. Ta różnica 0,46 px
   zostawiała po podmianie ciemną kreskę szerokości piksela wzdłuż starej krawędzi.
   Dla krawędzi wygładzonej powierzchniowo suma `(1 − pokrycie)` w oknie **jest**
   odległością do krawędzi — i to jest wielkość spójna z tym, co liczymy analitycznie.
3. **Wariant kontrolny.** Jeden „wariant" odtwarzający obecny kształt. Jeśli nie wychodzi
   różnica bliska zeru, model krawędzi albo barwy jest zły i nie ma sensu oglądać reszty.
   Ten test wyłapał oba błędy powyżej, zanim ktokolwiek zobaczył wynik.

## Skąd się biorą schodki na skośnej krawędzi

Osobny błąd tej samej rodziny, znaleziony w tym samym logotypie. Wcześniejsza naprawa
zapisywała pokrycie jako `clip(cel + 0.5 − y, 0, 1)` — rampę rozłożoną na **dokładnie
jeden piksel** w pionie. Dla krawędzi poziomej to jest poprawne. Dla krawędzi nachylonej
`m` px/px prawdziwe pokrycie rozkłada się na `1 + |m|` wierszy, więc rampa na jeden piksel
daje twarde, schodkowane przejście — dokładnie to, co człowiek widzi jako „kanciaste".

Lekarstwo jest jednolinijkowe: pokrycie licz **nadpróbkowaniem** (u mnie 8×8 na piksel)
zamiast wzorem na rampę. Nachylenie samo się wtedy uwzględnia, w każdym miejscu krawędzi
osobno, i nie trzeba go w ogóle mierzyć.

## Transferability

Działa wszędzie tam, gdzie trzeba poprawić kształt w gotowym rastrze: logotypy z generatora,
sprite'y, maski alfa, ikony UI. W Unity ta sama zasada dotyczy edycji tekstur maskujących
i kanałów alfa — zmieniaj pokrycie, nie kolor.

## Related
- [[20260811-1520-retusz-poswiaty-tylko-poza-sylwetka]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260811-1520-retusz-poswiaty-tylko-poza-sylwetka|Retusz poświaty wokół glifu — przemalowuj wyłącznie tło poza sylwetką]] - wspolne: obrobka-obrazu, numpy
<!-- /POWIAZANE:auto -->
