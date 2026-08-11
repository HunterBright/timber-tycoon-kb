---
title: Cena u posrednika to cena wybranego dostawcy, nie cena producenta
type: lesson
status: draft
confidence: high
verified: '2026-08-11'
date: 2026-08-11
project: GameDevOS
tags:
- radar
- pomiar
- falszywy-alarm
- agregatory
- ceny
applies_to:
- monitorowanie cen API
- kazdy detektor zmian czytajacy agregator zamiast zrodla
source: 'https://openrouter.ai/api/v1/models oraz https://api-docs.deepseek.com/quick_start/pricing'
severity: high
suggested-category: process/lessons
time_lost: '~40 min'
---

# Cena u posrednika to cena wybranego dostawcy, nie cena producenta

## Problem

Zbudowalismy 10.08.2026 narzedzie robiace codzienna migawke rejestru modeli
u posrednika (OpenRouter), zeby wylapywac ciche podwyzki cen. Drugiego dnia
zycia narzedzie zglosilo **dziewiec zmian ceny**, w tym dwie brzmiace jak
wydarzenie dnia:

- `deepseek-v4-pro`, wejscie i wyjscie: **drozej o 49,7%**
- `deepseek-v4-pro`, odczyt z pamieci podrecznej: **drozej o 1415,7%**

Pasowalo to do zapowiedzi: DeepSeek 06.08 ogloszil, ze planuje istotna
podwyzke. Raport dzienny mial wpisac to jako fakt.

Do tego doszlo piec zmian limitu dlugosci odpowiedzi, wygladajacych na
zmiane parametrow modeli.

## Root cause

**Cennik producenta u zrodla pokazywal tego samego dnia stare liczby co do
grosza** (0,435 / 0,87 dolara za milion), razem z niezmienionym zdaniem, ze
podwyzka dopiero *"jest planowana"*, bez daty i bez nowych stawek.

Rozstrzygnal dopiero odczyt listy dostawcow obslugujacych ten jeden model.
Posrednik nie ma jednej ceny za model. Ma **kilkanascie ofert od roznych
dostawcow** hostujacych te same wagi, a w polu najwyzszego poziomu wystawia
cene **tego dostawcy, do ktorego akurat kieruje ruch**:

- stara cena `0.000000435` nadal istniala, u dostawcy **DeepSeek**
- nowa cena `0.0000006512` nalezala co do cyfry do **Novita**

Nikt niczego nie podniosl. **Przelozylo sie trasowanie.** Ta sama diagnoza
objela oba modele Qwena (Alibaba na Parasail, Novita na SiliconFlow) oraz
**wszystkie piec zmian limitu dlugosci odpowiedzi**, bo limit tez jest
wlasnoscia oferty dostawcy, a nie modelu.

Bilans: **dwanascie alarmow, zero prawdziwych zmian u producenta.**

## Solution

Test rozstrzygajacy miesci sie w jednym zdaniu i nie wymaga zadnej wiedzy
o dziedzinie:

> **Jesli stara wartosc nadal gdzies istnieje, to nikt jej nie zmienil.
> Zmienil sie wybor.**

Dopiero gdy starej wartosci nie ma juz u zadnego dostawcy, a nowa u ktoregos
jest, mamy zmiane u zrodla.

W kodzie: przy wykryciu zmiany dopytujemy o liste ofert **tylko dla tego
jednego modelu** (nie dla wszystkich, wiec koszt jest zerowy w dniu bez
zmian) i przypisujemy wynik do jednej z trzech kategorii:

1. **zmiana u producenta** - starej wartosci nie ma juz nigdzie
2. **przeskok trasowania** - stara wartosc istnieje u poprzedniego dostawcy,
   nowa u innego; raport nazywa oba dostawcow z imienia
3. **nie sprawdzono** - listy ofert nie dalo sie odczytac

Trzecia kategoria jest rownie wazna jak dwie pierwsze. Stan niezmierzony nie
moze wygladac jak zmierzony.

## What didn't work

- **Porownanie samej liczby miedzy dniami.** Poprawne technicznie, bezuzyteczne
  znaczeniowo: mierzy koszt zapytania, a nie decyzje producenta.
- **Sprawdzenie tylko cennika producenta.** Pokazalo, ze podwyzki nie ma, ale
  nie wyjasnialo, skad wzielo sie dwanascie alarmow. Bez wyjasnienia mechanizmu
  wrocilyby jutro.
- **Sztuczne proby jednostkowe.** Przechodzily w komplecie, bo dotykaly tylko
  cen. Pierwszy przebieg na zywych danych wywalil sie natychmiast na wpisie
  o limicie odpowiedzi, ktory nie ma pola z nazwa ceny. **Sztuczna proba
  sprawdza, czy narzedzie umie wykryc to, co zaplanowales; nie sprawdza,
  czy to, co wykrywa, jest problemem.**

## Transferability

To nie jest lekcja o cenach ani o jednym posredniku. Ten sam ksztalt wystepuje
wszedzie, gdzie **czytamy warstwe posredniczaca zamiast zrodla, a warstwa
posredniczaca dokonuje wyboru**:

- rejestry pakietow wystawiajace ceny albo limity wielu hostow
- lustra repozytoriow z wlasnymi metadanymi (u nas: lustra wag modeli, ktore
  **gubily pole blokujace kraj**, przez co narzedzie z zakazem terytorialnym
  wygladalo na czyste)
- sieci dostarczania tresci i posrednicy zwracajacy inna zawartosc niz zrodlo
- agregatory sklepow i porownywarki
- warstwy tlumaczace jedno API na drugie

Regula ogolna: **przy kazdym detektorze zmian zapytaj, czy mierzysz rzecz,
czy czyjs wybor rzeczy.** Jesli miedzy Toba a zrodlem stoi cokolwiek, co moze
wybrac, detektor bedzie zglaszal cudze wybory jako zmiany swiata. Falszywy
alarm kosztuje tyle samo zaufania co przeoczenie: bramka, ktora krzyczy bez
powodu, zostaje wylaczona po tygodniu i wtedy nie zlapie tej jednej zmiany,
ktora naprawde ma znaczenie.

## Related

- [[20260810-0930-migawka-zamiast-zapytania-gdy-kanal-nie-ma-daty-zmiany]] - narzedzie, ktore ten blad popelnilo, i powod, dla ktorego powstalo
- [[20260810-1015-nieobecnosc-w-jednym-kanale-nie-jest-nieobecnoscia-w-swiecie]] - ta sama rodzina: poprawny pomiar, za szeroki wniosek
- [[20260808-0940-sprawdzian-ktory-nie-umie-pasc]] - dlaczego kazda poprawka potrzebuje kontroli dajacej INNY wynik

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-0930-migawka-zamiast-zapytania-gdy-kanal-nie-ma-daty-zmiany|Migawka zamiast zapytania, gdy kanal nie ma daty zmiany]] - wspolne: ceny, radar
<!-- /POWIAZANE:auto -->
