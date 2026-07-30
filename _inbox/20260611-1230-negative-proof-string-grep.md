---
title: 'Anti-pattern: dowodzenie "handler nie istnieje" grepem po stringu ID'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-06-11'
project: Timber_Tycoon
tags:
- diagnosis
- grep
- unlock-system
- data-driven
- negative-proof
- unity
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Anti-pattern: dowodzenie "handler nie istnieje" grepem po stringu ID

## Kontekst

Diagnoza martwego regału magazynowego (Timber Tycoon, 2026-06-10) stwierdziła:
"unlocki `spawn_*` nie mają ŻADNEJ obsługi w kodzie (grep: zero trafień)" i na tej
podstawie podjęto decyzję o włączeniu gate'owanych obiektów na stałe w scenie.

Wniosek był FAŁSZYWY. Handler istniał od 3 tygodni — ale był kluczowany po
**typie enuma** (`UnlockType.RackSpawn` + pole `targetId`), nie po stringu
`unlockId`. Grep po `spawn_` nie miał prawa go znaleźć, bo kod nigdy nie dotyka
tego stringa — czyta `entry.type` i `entry.targetId` z ScriptableObjecta.

## Anti-pattern

W systemach data-driven (unlocki, eventy, recepty w SO/JSON) identyfikator
rekordu często służy TYLKO do deduplikacji/zapisu, a dispatch idzie po typie,
kategorii lub osobnym polu target. Negatywny wynik grepa po ID dowodzi jedynie,
że nikt nie hardcoduje tego ID — NIE dowodzi braku obsługi rekordu.

Skutek wtórny: błędna "wiedza" trafiła do pamięci projektu i kolejna sesja
zaczęła pracę od fałszywej przesłanki — naprawiła scenę w kontrze do
zaimplementowanego (działającego!) systemu gatingu.

## Poprawny workflow

Zanim uznasz "nie ma obsługi X":
1. Otwórz definicję rekordu (klasę UnlockEntry/EventData itp.) i wypisz WSZYSTKIE
   pola: id, type, targetId, payload.
2. Grepuj po każdym z: nazwie typu enuma (`RackSpawn`), wartościach pól target
   (`ChipBagRack`), nazwie klasy rekordu (`UnlockEntry`) — nie tylko po id.
3. Przejdź łańcuch konsumpcji od miejsca, które ITERUJE po rekordach
   (np. `foreach (entry in level.unlocks)`), zamiast szukać od strony ID.
4. Negatyw potwierdzaj minimum dwiema niezależnymi drogami (np. grep po typie
   + breakpoint/log w pętli konsumującej).

## Reguła

> Grep po stringu ID to dowód poszlakowy. Dowodem braku handlera jest dopiero
> prześledzenie pętli, która konsumuje kolekcję rekordów tego typu.
