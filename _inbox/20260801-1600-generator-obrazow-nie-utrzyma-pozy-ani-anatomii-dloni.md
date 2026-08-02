---
title: Generator obrazow nie utrzyma pozy ani anatomii dloni - to nie jest zadanie dla referencji
type: lesson
status: draft
confidence: high
verified: 2026-08-01
tags: [generator-obrazow, referencja, npc, anatomia, t-poza, pipeline, qwen]
date: 2026-08-01
project: Kerf - Sawmill Tycoon
source: 'rodzina referencji NPC: 14 postaci, 58 obrazkow, Qwen-Image, 2026-08-01'
applies_to: [kazde tworzenie referencji postaci generatorem obrazow]
severity: high
time_lost: ok. 3 godziny generowania
---

# Generator obrazow nie utrzyma pozy ani anatomii dloni

## Zmierzone, nie odczute

Rodzina 14 postaci NPC w jednym stylu, 58 wygenerowanych obrazkow, opis
z twardym naciskiem na poze i liczbe palcow, do tego negatyw z zakazami.

| Co mierzone | Wynik |
|---|---|
| Czysta T-poza z przodu | **18 z 58** (31%) |
| Postacie z co najmniej jednym dobrym wariantem | 10 z 14 |
| Dlonie z poprawna liczba palcow | **mniejszosc**; czesto plaskie lopatki bez podzialu |

Cztery postacie **nie dostaly poprawnej pozy w zadnym z czterech wariantow**,
przy dwoch roznych ziarnach i trzech przebiegach. Dosypywanie generacji
tego nie naprawia - to nie jest pech, tylko sufit narzedzia.

## Dlaczego dlonie nie moga wyjsc

W kadrze 1024 px na cala sylwetke **dlon zajmuje okolo 60 pikseli**.
Piec osobnych palcow tam sie nie miesci. Zadne wzmocnienie opisu
("FIVE digits - FOUR fingers plus ONE thumb") ani negatywu
("four fingers, three fingers, fused fingers, mitten hands") tego nie zmieni,
bo problem jest w rozdzielczosci, nie w opisie.

## Wniosek: podzial rol

> **Referencja z generatora obrazow jest zrodlem STYLU, UBIORU i KOLORU.
> Poza i anatomia biora sie z wlasnego generatora postaci, gdzie sa sterowane
> liczbami.**

To jest ten sam podzial, ktory juz raz ustalilismy dla proporcji: opisem
nie da sie wymusic 7,5 glowy, wiec proporcje wpisuje sie w Blenderze.
Dlonie i poza naleza do tej samej kategorii.

Potwierdzenie z wlasnego dorobku: dlonie robione osobno w generatorze
Blendera (`KerfPostac\dlon\DLON_koncowy.png`) sa poprawne - cztery palce
plus kciuk, czysta siatka low poly. Walka o poprawna dlon w obrazku
referencyjnym jest walka o cos, czego i tak sie nie uzyje.

## Tansza droga do kompletu poz

Zamiast losowac poze przy kazdej nowej postaci, wziac **jedna zatwierdzona
sylwetke w T-pozie i przebierac ja** narzedziem do edycji obrazu (Qwen-Image-Edit).
Poza jest wtedy DZIEDZICZONA, a nie losowana. Ta sama technika przy widokach
robotnika dala zgodnosc odcieni 8 na 9 elementow w granicach 12/255.

## Bramka, ktora z tego powstala

`D:\AI\filtr_tpoza.py` - mierzy sylwetke i odrzuca wszystko, co nie jest czysta
T-poza z przodu. Ma udowodniony tryb porazki (rysuje T i A, sprawdza rozroznienie).

**Znalazl w sobie dziure w trakcie pracy:** przepuszczal obrazek z trzema
postaciami, bo trzy sylwetki obok siebie daja szeroki obrys na wysokosci barkow
i test rozpietosci przechodzil. Poprawka: liczenie ODDZIELNYCH bryl na wysokosci
tulowia. Jedna postac to jedna bryla.

Lekcja w lekcji: **bramka sprawdzajaca proporcje obrysu nie sprawdza, ILE jest
obiektow.** Przy kazdej mierze ksztaltu warto osobno policzyc liczbe rzeczy.

## Powiazane

- [[20260725-1830-image-model-cannot-force-figure-proportions]]
- [[gate-must-have-provable-failure-mode]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-1330-liczbe-postaci-mow-twierdzaco-negatyw-nie-wystarcza|Liczbe postaci w kadrze trzeba powiedziec TWIERDZACO - negatyw jej nie pilnuje]] - wspolne: generator-obrazow, qwen, referencja
- [[pipeline-style-npc-spawn|Pipeline-Style NPC Spawn (OnPurchaseComplete Trigger)]] - wspolne: pipeline, npc
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] - wspolne: pipeline, npc
<!-- /POWIAZANE:auto -->
