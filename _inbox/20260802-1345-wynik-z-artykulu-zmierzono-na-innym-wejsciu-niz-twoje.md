---
title: Wynik z artykulu zmierzono na innym wejsciu niz twoje - czytaj zgloszenia, nie README
type: lesson
status: draft
confidence: high
verified: '2026-08-02'
tags:
- narzedzia-ai
- generowanie-3d
- retopologia
- ocena-narzedzi
- due-diligence
date: 2026-08-02
project: GameDevOS
source: 'https://github.com/LoHhhha/LATO.2/issues/2'
applies_to: []
severity: high
time_lost: ''
---

# Wynik z artykulu zmierzono na innym wejsciu niz twoje

## Objaw

LATO.2 to narzedzie do retopologii, ktore obiecuje sprowadzenie siatki do zadanej
liczby wierzcholkow. Artykul podaje jakosc odwzorowania (IoU) **0,9603**. Licencja
czysta (MIT), kod dostepny, autorzy uczciwie ostrzegaja o dziurach w siatce.
Na tej podstawie zapisalismy je jako "brakujace ogniwo miedzy generatorem
a szkieletem" i wpisalismy na liste do wdrozenia.

Dzien pozniej okazalo sie, ze na siatkach **dolaczonych do samego repozytorium**
uzytkownik zmierzyl IoU **0,02**. Czyli nie o kilka procent gorzej, tylko
kompletnie inaczej.

## Przyczyna

Odpowiedz autora narzedzia wyjasnila mechanizm, cytat doslowny:

> "crocodile.glb and the other meshes you fed into the model are not 'artist
> meshes' but AI-generated 'dense mesh' which is not used for evaluation."

I doprecyzowanie: *"We used artist mesh(from Shapenet/Objaverse/Toys4k/...)
when testing V-VAE."*

Narzedzie liczono na siatkach zrobionych przez modelarza. Nasze wejscie to
z definicji gesta siatka prosto z generatora AI (TRELLIS, Hunyuan, Tripo).
To sa dwie rozne klasy geometrii i narzedzie zachowuje sie na nich zupelnie
inaczej.

Sedno jest takie: **liczba w artykule nie byla falszywa, tylko zmierzona
na innym wejsciu niz nasze**. Nikt nie klamal. Po prostu zbior testowy
autora i nasz potok produkcyjny to dwa rozne swiaty, a zaden dokument
narzedzia tego nie mowil.

## Rozwiazanie

Przed przyjeciem narzedzia AI do potoku sprawdz **na czym je mierzono**,
a nie tylko **ile wyszlo**:

1. Znajdz w artykule nazwe zbioru testowego. Jesli to ShapeNet, Objaverse,
   Toys4k albo inny zbior modeli robionych przez ludzi, a twoje wejscie
   pochodzi z generatora, **wyniki nie przenosza sie automatycznie**.
2. **Przeczytaj otwarte zgloszenia w repozytorium.** To jest jedyne miejsce,
   gdzie widac, jak narzedzie zachowuje sie na cudzych danych. README pokazuje
   najlepszy przypadek, zgloszenia pokazuja rozklad.
3. Szukaj w zgloszeniach zwrotow "cannot reproduce", "unable to reproduce",
   "different results", "bad quality". Zwroc uwage, czy autor odpowiedzial
   i czy podal przepis na przygotowanie wejscia.
4. Zanim wydasz godzine na instalacje, uruchom narzedzie **na wlasnej,
   typowej siatce**, a nie na przykladach z repozytorium. Przyklady sa dobrane.

## Co NIE zadzialalo

Sprawdzenie licencji, jakosci kodu, liczby gwiazdek i README. Wszystkie cztery
wyszly dobrze i wszystkie cztery byly nieistotne dla pytania "czy to zadziala
u nas". Zaden z tych sygnalow nie dotykal jedynej rzeczy, ktora przesadzila.

Nie zadzialalo tez czekanie na "pierwszy niezalezny test" jako osobny krok:
test istnial od dziesieciu dni, tylko lezal w zakladce ze zgloszeniami,
a nie w formie wpisu na blogu czy filmu.

## Dowod

Zgloszenie https://github.com/LoHhhha/LATO.2/issues/2, otwarte 22.07.2026,
trzy komentarze, **status otwarty na 02.08.2026**. Cytaty wyzej pochodza
z interfejsu programistycznego GitHuba, nie z cudzego streszczenia.
Autor nie podal przepisu na przygotowanie wejscia, mimo ze o niego proszono.

## Czy to przeniesie sie na inny projekt

Tak, i to szeroko. Dotyczy kazdego narzedzia opartego na uczeniu maszynowym,
ktore wciaga sie do potoku produkcyjnego: generowanie 3D, retopologia,
rigowanie, przenoszenie ruchu, upscaling tekstur, generowanie dzwieku.
Wszedzie tam liczba z artykulu opisuje zachowanie na zbiorze testowym autora,
a nie na tym, co wychodzi z twojego poprzedniego kroku.

Regula, ktora z tego zostaje: **im dluzszy potok narzedzi AI, tym wieksza szansa,
ze wyjscie kroku N nie przypomina niczego, na czym mierzono krok N plus jeden.**
Najbardziej zdradliwe sa wlasnie miejsca styku, bo kazde narzedzie z osobna
dziala poprawnie.

## Powiazane

- [[20260802-1120-licencja-repo-github-to-nie-licencja-wag-na-hugging-face]]
- [[20260801-1245-regulamin-uslugi-a-licencja-wag-to-dwa-rozne-swiaty]]
