---
type: pattern
project: Discord_Studio
suggested-category: tooling/patterns
tags:
- grafika
- skalowanie
- logo
- sdf
- python
- pipeline-assetow
date: 2026-08-10
status: draft
---

# Powiększanie logo bez rozmycia: pole odległości ze znakiem

## Problem, który rozwiązuje

Logo wygenerowane przez model obrazu ma rozdzielczość, jaką ma — u nas 1280x720. Do intra gry
w 4K trzeba go powiększyć trzykrotnie. Zwykłe skalowanie (Lanczos, bicubic) rozmywa krawędzie
liter i logo wygląda na tanie. Modele powiększające typu ESRGAN wymagają pobrania i potrafią
dorysować szczegóły, których nie było.

## Spostrzeżenie, na którym stoi cała metoda

W obrazie logo są dwa rodzaje informacji i **skalują się zupełnie inaczej**:

- **Krawędzie kształtów** — nieciągłe. Skalowanie je rozmywa.
- **Gradienty i poświata** — gładkie z natury. Skalowanie ich nie psuje.

Więc: krawędzie odtworzyć od nowa, kolory po prostu przeskalować.

## Metoda

1. **Maska kształtu.** Progowanie jasności. Jeden próg dla wszystkich liter; rozdzielanie na
   „ciepłe" i „chłodne" rozrywa litery o barwie pośredniej (u nas kremowy napis STUDIO wpadał
   raz do jednej maski, raz do drugiej, i wychodził dziurawy).
2. **Pole odległości ze znakiem.** `distance_transform_edt` na masce i jej negatywie,
   odjąć od siebie. Wewnątrz kształtu wartości ujemne, na zewnątrz dodatnie.
3. **Korekta podpikselowa — to jest sedno.** Samo pole z maski zero-jedynkowej daje
   po powiększeniu **schodki**, bo krawędź jest zaokrąglona do całych pikseli. Ale piksel na
   krawędzi ma w oryginale pośrednią jasność, która mówi, w jakim ułamku jest pokryty
   kształtem. Z tego liczy się dokładne położenie krawędzi: `sdf = 0.5 - alpha` dla pikseli
   przy krawędzi. Bez tego kroku metoda nie ma sensu.
   Pokrycie liczyć **lokalnie** (filtr max/min w oknie ~9 px), a nie przez erozję maski —
   erozja zjada cienkie kreski i psuje pomiar.
4. **Powiększenie pola** interpolacją dwusześcienną, pomnożone przez współczynnik skali
   (odległości rosną razem z obrazem).
5. **Render krawędzi**: `pokrycie = clip(0.5 - pole, 0, 1)`.
6. **Kolory**: powiększyć oryginał Lanczosem, a następnie **rozciągnąć kolor wnętrza na
   krawędź** — inaczej krawędź weźmie piksele zanieczyszczone tłem. Robi to
   `distance_transform_edt(..., return_indices=True)`: każdemu pikselowi przypisz kolor
   najbliższego piksela z wnętrza kształtu. To samo osobno dla tła.
7. **Złożenie**: `kolor_litery * pokrycie + kolor_tla * (1 - pokrycie)`.

## Ograniczenie, o którym trzeba wiedzieć

Na **cienkich kreskach metoda szumi** — korekta podpikselowa czyta jasność z kilku pikseli,
a przy kresce grubości 8 px stosunek sygnału do szumu jest kiepski i krawędź faluje.
Rozwiązanie: podzielić obraz na pasy i cienki tekst zostawić klasycznemu Lanczosowi.
Szew poprowadzić przez obszar czystego tła, wtedy jest niewidoczny.

## Kanał alfa

Przy okazji da się zrobić wersję z przezroczystością, ale **poświatę trzeba wygenerować od
nowa** z maski kształtu (rozmycie gaussowskie w dwóch promieniach), a nie wyciągać z obrazu
przez odejmowanie tła. Wyciąganie przepuszcza do kanału alfa winietę i szum, przez co na
kolorowym tle widać prostokątną ramkę. Na koniec zbić szum progiem:
`alpha = clip((alpha - 0.05) / 0.95, 0, 1)`.

## Kiedy tego NIE używać

Gdy logo istnieje w wektorze — wtedy po prostu wyrenderować z wektora. Ta metoda jest dla
sytuacji, w której ma się wyłącznie rastrowy oryginał i nie chce się go przerysowywać.

## Related
- [[20260810-2140-model-edytujacy-psuje-to-czego-nie-dotyka]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-2140-model-edytujacy-psuje-to-czego-nie-dotyka|Model edytujący psuje elementy, o które nie proszono - do drobnych poprawek użyć pikseli]] - wspolne: logo, grafika, pipeline-assetow
<!-- /POWIAZANE:auto -->
