---
title: Odzysk dokumentu z PDF - pass rekonstrukcji zamiast surowego pdftotext
type: pattern
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: Another Quest
tags:
- dokumentacja
- pdf
- odzysk
- workflow
applies_to: []
source: 'noc D1 sprintu AQ, job [S2b] - odzysk symulacji 30 min ze 152-stronicowego eksportu rozmowy'
suggested-category: workflow/patterns
---

# Odzysk dokumentu z PDF - pass rekonstrukcji zamiast surowego pdftotext

## When to use

Gdy jedyna kopia waznego dokumentu (decyzje, audyt, specyfikacja) zyje w eksporcie rozmowy z modelu
albo w cudzym PDF-ie, a ma trafic do repo jako zrodlo, do ktorego beda sie odwolywac nastepne sesje.

## Steps

1. `pdftotext -enc UTF-8 -layout plik.pdf wyjscie.txt` - `-layout` trzyma tabele w kupie lepiej niz tryb
   domyslny, ale ich NIE naprawia.
2. Znajdz granice dokumentu w eksporcie (`grep -n` po naglowku i po zdaniu koncowym) - eksport rozmowy
   zawiera tez wszystko dookola, czego nie chcesz.
3. **Pass rekonstrukcji**, w tej kolejnosci:
   - tabele skladasz od nowa i **weryfikujesz matematycznie**: kazda liczba musi zgadzac sie z wyliczeniem
     w sasiedniej kolumnie. To jest jedyny mechaniczny test, jaki masz;
   - usuwasz artefakty stron (stopki, naglowki, sztuczne lamania akapitow);
   - sprawdzasz diakrytyki (warstwa tekstowa czesto je gubi).
4. Naglowek pliku mowi **wprost "Rekonstrukcja, nie verbatim"**, podaje zrodlo, date oryginalu i **liste
   tego, co bylo skladane od nowa** oraz zdanie "zadna liczba ani wniosek nie zostaly zmienione".
5. Zachowujesz oryginalna numeracje znalezisk/sekcji - stan maszynowy i przyszle notatki beda sie na nia
   powolywac.
6. PDF zrodlowy ladujesz do `archive/` (przez LFS) i **nie kasujesz oryginalu w folderze snapshotu**.

## Why this works

Tabela w PDF-ie nie ma struktury - ma wspolrzedne. Warstwa tekstowa zwraca komorki w kolejnosci rysowania,
wiec wartosci laduja w cudzych wierszach i wyglada to na poprawny dokument. Weryfikacja matematyczna
jest jedynym sposobem, zeby to zlapac bez otwierania PDF-a obok. Naglowek "rekonstrukcja" chroni przed
gorszym problemem: dokument odzyskany bez tej noty jest za rok nieodroznialny od oryginalu i nikt juz
nie sprawdzi, ktora liczba jest z zrodla, a ktora ze skladania.

## Trade-offs

Kosztuje realny czas (przy 550 wierszach: ~30 min) i wymaga, zeby ktos rozumial tresc na tyle, by
zweryfikowac liczby. Nie da sie tego zdelegowac do "przepisz mi to".

## Variants

Jesli dokument nie ma tabel ani liczb, pass rekonstrukcji redukuje sie do usuniecia stopek - ale nota
"rekonstrukcja, nie verbatim" zostaje, bo zdania tez potrafia sie posklejac z dwoch kolumn.
