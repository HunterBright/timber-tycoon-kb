---
title: Kanoniczny hash wyniku jako dowod neutralnosci refaktoru generatora
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- refactoring
- procedural-generation
- regression
- hashing
- blender
- asset-pipeline
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Kanoniczny hash wyniku jako dowod neutralnosci refaktoru generatora

## Problem it solves

Masz skrypt generujacy asset (siatke, teksture, teren, atlas). Wynik jest ZAAKCEPTOWANY,
zacommitowany i uzywany w grze. Teraz musisz przebudowac generator - sparametryzowac go,
rozbic na moduly, dodac drugi wariant.

Pytanie, ktore trzeba umiec rozstrzygnac: **czy po refaktorze wychodzi DOKLADNIE ten sam
asset?** "Wyglada tak samo" nie jest odpowiedzia - roznica milimetrowa nie jest widoczna,
a i tak psuje zaakceptowany model.

## Solution

Przed dotknieciem czegokolwiek policz **kanoniczny hash wyniku** i zapisz go jako wzorzec.
Po kazdym kroku refaktoru policz ponownie i porownaj.

Kanoniczny = niezalezny od kolejnosci, w jakiej narzedzie trzyma dane, i od szumu
zmiennoprzecinkowego:

- kolejnosc obiektow ustalona z gory (lista nazw), nie z kolejnosci w pliku
- kolejnosc wierzcholkow/scianek po INDEKSIE
- wspolrzedne kwantyzowane (u nas 1e-6 m przy modelu 4 m = 0.25 ppm)
- **zero ze znakiem sprowadzone do zwyklego zera** - inaczej `-0.0` i `0.0` daja rozne hashe
  przy identycznej geometrii
- do hasha wchodzi WSZYSTKO, co ma znaczenie: hierarchia, transformy, indeksy materialow,
  nazwy slotow, kolory per naroznik (nie per sciana - zlapie tez niejednolite malowanie)

Do tego raport z liczbami (wierzcholki, scianki, per-obiekt skrocone hashe), zeby rozjazd
dalo sie PRZECZYTAC, a nie tylko zobaczyc "inny hash".

## Krok zerowy, bez ktorego reszta jest bezwartosciowa

**Najpierw udowodnij, ze generator jest deterministyczny.** Policz hash dwa razy na
nietknietym pliku, potem SKASUJ plik, przebuduj od zera i policz trzeci raz. Trzy identyczne
= mozna isc dalej. Jesli generator nie jest deterministyczny (kolejnosc slownikow, losowy
seed, znacznik czasu), caly dowod neutralnosci nie ma sensu - a dowiadujesz sie o tym w 10
minut zamiast po polowie refaktoru.

## Czego NIE hashowac

**Nie porownuj bajtow pliku wyeksportowanego.** Eksportery zapisuja w naglowku znacznik
czasu (FBX: `CreationTimeStamp`), wiec dwa eksporty TEJ SAMEJ geometrii roznia sie bajtami.
Taki test dawalby falszywy alarm przy kazdym uruchomieniu i po tygodniu zaczalby byc
ignorowany - czyli bylby GORSZY niz brak testu.

Zrodlem prawdy jest plik zrodlowy generatora (u nas `.blend`), nie artefakt eksportu.
Jesli chcesz zweryfikowac sam eksport, zrob to osobno i **eksportuj na bok** (do katalogu
tymczasowego), zeby nie brudzic zaakceptowanego assetu w repozytorium.

## Consequences

- Refaktor przestaje byc aktem wiary. U nas: rozbicie 600-liniowego skryptu na trzy moduly,
  przeniesienie kilkunastu tabel i zmiana sposobu adresowania pasm - hash identyczny po
  kazdym kroku.
- Dostajesz tanie zabezpieczenie na przyszlosc: kazda pozniejsza zmiana wspolnego kodu
  konczy sie jednym poleceniem sprawdzajacym, czy stare warianty nie ucierpialy.
- Koszt: okolo 150 linii jednorazowego skryptu.
- Ograniczenie: hash mowi TYLKO "identyczne / rozne". Do oceny, czy roznica jest akceptowalna,
  potrzebny jest raport z liczbami obok hasha.

## Zasada pochodna: wyprowadzaj w przod, nie wstecz

Przy refaktorze kuszace jest zamienic wpisane na sztywno liczby (pozycje detali) na wzory
wyprowadzone z kotwic. **Nie rob tego na wariancie, ktory ma zacommitowany wzorzec** - kazdy
wzor trafi o milimetry obok i zlamie hash bez zysku. Wzory wprowadzaj w NOWYCH wariantach,
ktore nie maja czego regresowac. Stary wariant zostaje przy literalach.

## Related

- [[20260720-1306-walidator-spelniony-przez-konstrukcje]] - hash odpowiada "czy sie zmienilo", walidatory
  odpowiadaja "czy jest poprawne". Potrzebne oba; hash nie zastapi walidatorow.
