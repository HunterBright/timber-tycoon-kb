---
title: Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-12'
project: Kerf - Sawmill Tycoon
tags:
- save-system
- migration
- versioning
- unity
- isaveable
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu

## Kontekst
Zmiana domyślnej wartości w progresji (np. salon mebli: startowa liczba slotów 3 -> 2, gdzie 3. slot staje się płatnym ulepszeniem `pedestal_03`). Trzeba było ochronić STARE zapisy (gdzie gracz miał 3 sloty za darmo), nie psując nowego defaultu ("start od 2") dla nowych gier.

## Anti-pattern (naiwna migracja)
```
// ZLE: przy KAZDYM wczytaniu
if (IsUnlocked("showroom_base") && !IsUnlocked("pedestal_03"))
    Grant("pedestal_03");   // "grandfather" 3. slot
```
Ten warunek jest prawdziwy dla KAZDEGO zapisu, w ktorym gracz ma bazowy unlock ale swiadomie NIE kupil nowego platnego ulepszenia. Skutek: gracz w NOWEJ grze kupuje salon (2 sloty), zapisuje, wczytuje -> dostaje 3. slot za darmo. Migracja permanentnie kasuje nowy default "start od 2" dla wszystkich - jest nieodroznialna od "grandfather starego zapisu".

## Poprawka (bramka wersją schematu)
Dodaj pole `schemaVersion` do DTO zapisu. Zapis ustawia je na wersję bieżącą. Migracja jednorazowa działa TYLKO gdy wczytany schemat jest starszy:
```
if (data.schemaVersion < 1 && IsUnlocked("showroom_base") && !IsUnlocked("pedestal_03"))
    Grant("pedestal_03");   // odpala sie RAZ na zapis sprzed zmiany
// GetSaveData zapisuje schemaVersion = CURRENT (1) -> po pierwszym zapisie migracja juz nie odpali
```
Zapisy sprzed zmiany (schemaVersion=0, domyslne int przy braku pola = wsteczna kompatybilnosc JsonUtility) migrują raz; nowe gry (schemaVersion>=1) nigdy.

## Zasada ogolna
Kazda migracja "dogrywajaca" stan przy wczytaniu, ktora zmienia domyslne zachowanie, musi byc jednorazowa i gatowana MONOTONICZNIE ROSNACA wersja schematu zapisu - nie stanem, ktory legalna nowa gra tez moze miec. "Czy ten warunek jest prawdziwy takze dla swiezego zapisu z nowym defaultem?" Jesli tak -> potrzebujesz wersji schematu.
