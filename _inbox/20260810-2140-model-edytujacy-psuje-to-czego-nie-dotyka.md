---
title: Model edytujący psuje elementy, o które nie proszono - do drobnych poprawek użyć pikseli
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-10
project: Discord_Studio
tags:
- comfyui
- qwen-image-edit
- grafika
- logo
- pipeline-assetow
applies_to: []
source: 'Poprawka litery G w logotypie MGDB Studio'
severity: medium
suggested-category: tooling/lessons
time_lost: '~30 min'
---

# Model edytujący psuje elementy, o które nie proszono

## Problem

Gotowy logotyp miał jedną wadę: dolna krawędź litery G zapadała się o około jeden piksel,
tworząc widoczny uskok. Zadanie brzmiało „popraw tylko G, reszty nie ruszaj".

Qwen-Image-Edit 2509 dostał wyraźne polecenie zachowania wszystkiego poza jedną literą.
Przy pełnej sile edycji (denoise 1.0) **przerysował sąsiednią literę D na bezkształtną plamę**.
Przy sile 0.55 zostawił układ, ale wyciął w D ścięcie, którego wcześniej nie było.
Przy sile 0.38 zachował obraz idealnie — i **nie naprawił G**, bo wady w ogóle nie zauważył.

Zależność jest monotoniczna i nie ma w niej okna, w którym model naprawia jedno, nie psując
drugiego: siła potrzebna do zmiany czegokolwiek jest większa niż siła, przy której zaczyna
zmieniać wszystko.

## Root cause

Model edytujący nie ma pojęcia „obszar zainteresowania". Przetwarza cały obraz przez latent
i odtwarza go w całości. Polecenie „nie zmieniaj reszty" jest sugestią dla warstwy tekstowej,
a nie ograniczeniem operacji. Im więcej szumu wprowadzi, tym więcej odtworzy po swojemu.

Dodatkowo wada rzędu jednego piksela jest poniżej progu, na którym model w ogóle rozpoznaje
„błąd" — dla niego to szum, a nie usterka kształtu.

## Solution

**Do wad mniejszych niż kilka pikseli: operacja na pikselach, nie generatywna.**

Zadziałało podejście dwustopniowe w Pythonie (PIL + numpy):

1. **Zmierzyć wadę liczbowo, nie na oko.** Dla każdej kolumny znaleźć dolną krawędź kształtu
   progiem jasności i wypisać profil. Zafalowanie 426 → 422 → 423 widać w liczbach od razu,
   a na obrazie łatwo je pomylić z zamierzonym łukiem.
2. **Dopełniać, nigdy nie kasować.** Wyznaczyć gładki łuk odniesienia średnią kroczącą i dodać
   piksele tylko tam, gdzie krawędź jest wyżej niż łuk. Kasowanie pikseli natychmiast tworzy
   nowe artefakty — pierwsza wersja skryptu przerysowywała całą krawędź i zrobiła czarny klin
   w rogu oraz schodki.
3. **Wygaszać maskę na brzegach obszaru.** Bez rampy na końcach zakresu powstaje szew widoczny
   bardziej niż naprawiana wada.
4. **Smugę wewnątrz kształtu wygładzać w osi prostopadłej do gradientu.** Litera miała gradient
   pionowy, więc uśrednianie poziome usunęło smugę i nie ruszyło gradientu. Filtr medianowy
   albo rozmycie w obu osiach zniszczyłyby przejście tonalne.

Efekt: zmienione 0,16% pikseli obrazu, reszta bit w bit identyczna.

## What didn't work

- **Proszenie modelu o zachowanie reszty.** Bez skutku na każdej sile edycji.
- **Zjeżdżanie z denoise.** Albo psuje, albo nie naprawia. Nie ma progu pośredniego.
- **Wykrywanie wady jako „dziury" progiem jasności.** Ciemna smuga wewnątrz kształtu miała
  jasność powyżej progu tła, więc analiza przerw jej nie widziała. Trzeba było jej szukać
  jako odchylenia od gradientu, nie jako braku pikseli.

## Transferability

Dotyczy każdego projektu, w którym generatory obrazu produkują materiały wymagające dokładności:
logo, ikony, tekstury z powtarzalnym wzorem, elementy interfejsu. Reguła praktyczna:

**Generator do tworzenia kształtu, kod do jego poprawiania.** Jeśli wada mieści się w kilku
pikselach, model jej nie widzi, a przy próbie naprawy zniszczy sąsiedztwo. Napisanie
dwudziestu linii operujących na tablicy pikseli jest szybsze i daje wynik powtarzalny,
czego generator z definicji nie zapewnia.

## Related
- [[generatory-3d]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-2320-skalowanie-logo-polem-odleglosci|20260810-2320-skalowanie-logo-polem-odleglosci]] - wspolne: logo, grafika, pipeline-assetow
- [[20260809-2130-ikony-jeden-przedmiot-zamiast-sceny|20260809-2130-ikony-jeden-przedmiot-zamiast-sceny]] - wspolne: grafika, comfyui
<!-- /POWIAZANE:auto -->
