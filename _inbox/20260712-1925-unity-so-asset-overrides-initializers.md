---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, scriptableobject, serialization, config, gotcha, tuning]
date: 2026-07-12
status: draft
---

# ScriptableObject .asset serializuje wartosci i WYGRYWA nad inicjalizatorami pol w C#

## Severity: medium (cicha regresja - kod wyglada na zmieniony, runtime uzywa starych wartosci)

## Problem
Config gry trzymany jako ScriptableObject (`public float qualityMultFine = 1.35f;`) ladowany
przez `Resources.Load<T>(...)`. Zmiana wartosci inicjalizatora w pliku `.cs` NIE zmienia zachowania
w grze, bo istniejacy asset `.asset` ma juz zserializowana STARA wartosc, ktora nadpisuje
inicjalizator przy deserializacji. Efekt: refaktor "dziala" (kompiluje sie), ale ekonomia/tuning
zostaja po staremu - regresja bez zadnego bledu.

## Dlaczego
Unity serializuje pola publiczne/[SerializeField] SO do pliku `.asset` (YAML). Przy `Resources.Load`
tworzony jest obiekt z inicjalizatorami, po czym deserializacja NADPISUJE pola wartosciami z pliku.
Inicjalizator w kodzie to tylko wartosc domyslna dla NOWEGO, jeszcze niezapisanego assetu.

## Rozwiazanie
Przy zmianie wartosci istniejacego SO-configu zmienic je W DWOCH miejscach:
1. inicjalizator w `.cs` (dla nowych assetow / fallbackow gdy asset nie istnieje),
2. odpowiadajacy klucz w pliku `.asset` (YAML to plain text - mozna edytowac recznie/skryptem),
   ALBO usunac klucz z YAML, zeby zadzialal inicjalizator.

Nowe pola dodane do klasy, ktorych NIE ma w `.asset`, poprawnie biora wartosc z inicjalizatora
(brak klucza = brak nadpisania) - te nie wymagaja edycji assetu.

## Jak wykryc
Objaw: "zmienilem stala w configu, a w grze bez zmian". Otworz `.asset` w edytorze tekstu i sprawdz,
czy klucz tam jest z inna wartoscia. Klucze nieznane klasie (usuniete pola) sa ignorowane po cichu.

## Case study (Timber Tycoon)
Refaktor cen mebli: zmiana `qualityMultFine`/`qualityMultMasterwork`/`reputationDivisor` w
`FurnitureConfig.cs` nie skutkowala - `Resources/FurnitureConfig.asset` mial stare 1.35/1.8/50.
Fix: reczna edycja YAML assetu. Nowo dodana drabinka cen desek dzialala od razu (brak w YAML).
