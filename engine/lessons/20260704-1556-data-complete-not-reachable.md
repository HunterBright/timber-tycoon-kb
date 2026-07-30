---
title: '"Wszystkie assety gotowe" ≠ grywalne - zawsze prześledź ŚCIEŻKĘ ZDOBYCIA, nie tylko obecność danych'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- content-pipeline
- gameplay-reachability
- economy
- verification
- dead-code
applies_to:
- unity
- tycoon
- any-content-game
source: ''
severity: high
time_lost: ''
promoted: '2026-07-30'
---

# "Wszystkie assety gotowe" ≠ grywalne - zawsze prześledź ŚCIEŻKĘ ZDOBYCIA, nie tylko obecność danych

## Problem
Zadanie brzmiało „dokończ brakujące modele towarów + ikony 6 nowych gatunków". Recon pokazał, że modele/ikony/prefaby/ItemData/dane cenowe/wpisy w bazie drewna dla wszystkich 6 gatunków BYŁY już gotowe (zrobione wcześniej). Mimo to gatunki były w praktyce **100% nieosiągalne** - gracz nie mógł dotknąć ani jednego. Gdyby przyjąć „dane są → gotowe", zbudowałoby się duplikaty gotowych rzeczy i ani jeden gatunek dalej nie byłby grywalny.

## Root cause
Dwa niezależne mechanizmy „martwej, ale kompletnej treści":
1. **Zamknięta pętla zdobywania.** Jedyne źródło sadzonki gatunku = ścięcie DOROSŁEGO drzewa tego gatunku. Żadne dorosłe drzewo 6 gatunków nie było postawione na mapie, brak sklepu z sadzonkami → pętla nigdy się nie startuje. Kompletny mebel bez drzwi.
2. **Martwe systemy równoległe.** Istniały DWA systemy receptur i DWA systemy cen; ten „oczywisty do uzupełnienia" (`Recipe_Sawmill_*`, `Market_*`) był martwy (zero runtime callerów) - żywy był inny (`CustomerManager.orderableItems`). Uzupełnianie martwego = praca w próżnię.

## Solution
- Zanim uznasz treść za „gotową", **prześledź ścieżkę gracza end-to-end**: zdobycie surowca → przetworzenie → sprzedaż/użycie → (odnowienie). Każde ogniwo musi mieć realny mechanizm, nie tylko dane.
- Dla każdego „systemu do uzupełnienia" **grep po runtime callerach** (`FindObjectsByType`, `.Instance`, referencje w scenie) ZANIM go rozszerzysz - potwierdź, że jest żywy.
- Konkretny fix tutaj: postawić dorosłe (ścinalne) drzewa na mapie (odblokowuje całą pętlę, bo systemy w dół już obsługiwały wszystkie gatunki) + dodać produkty do ŻYWEJ puli sprzedaży, nie do martwych assetów.

## What didn't work
- Wstępny plan „dorób receptury tartaku (`Recipe_Sawmill_*`) i ceny rynkowe (`Market_*`)" - oba to martwy kod; efekt w grze = zero. Uratował recon (grep callerów), nie intuicja z nazw plików.

## Transferability
Dotyczy KAŻDEJ gry z treścią odblokowywaną/zdobywaną (tycoon, RPG, crafting, roguelike, kolekcje). „Asset istnieje w projekcie" i „gracz może to zdobyć/użyć" to dwa różne stany - często rozjechane po tym, jak ktoś zbuduje dane, a integrację odłoży („DEFERRED"). Reguła: definicja „done" dla treści = przejście całej pętli zdobycia w rozgrywce, nie zielony ptaszek przy assetach. Oraz: przy wielu systemach robiących „to samo" zawsze potwierdź, który jest podłączony, zanim go rozbudujesz.

## Related
- (project memory) project_new_species_playable_2026-07-04
- (project memory) project_orderable_pool_owner - żywa pula sprzedaży vs martwe Market_*
