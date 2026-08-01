---
title: Klucz zapisu z hasha ścieżki NAZW = kolizja przy duplikatach obiektów
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-system
- hash
- collision
- isaveable
applies_to: []
source: ''
severity: high
promoted: '2026-07-30'
---

# Klucz zapisu z hasha ścieżki NAZW = kolizja przy duplikatach obiektów

## Objaw
`SaveManager: zduplikowany klucz zapisu 'tree_297C9595' - dwa systemy współdzielą klucz,
jeden nadpisze drugiego przy wczytaniu`. Stan jednego z dwóch obiektów ginął po load.

## Przyczyna
Fallbackowy klucz zapisu liczony jako `hash(ścieżka-nazw-w-hierarchii)`. Ścieżka zawiera
TYLKO nazwy GameObjectów, więc dwa obiekty o tej samej nazwie pod tak samo nazwanym rodzicem
(duplikat w scenie, dwa `Prefab(Clone)` pod wspólnym parentem) dają IDENTYCZNY string ->
identyczny hash -> identyczny klucz. Do tego hash 31-bitowy, więc kolizje możliwe nawet
przy różnych ścieżkach.

## Zasada
Klucz zapisu NIGDY z samej ścieżki nazw. Minimum: dołóż `GetSiblingIndex()`; najlepiej
trwały GUID per instancja (serializowane pole; uwaga - dla istniejących obiektów w scenie
binarnej to wymaga edycji sceny). Duplikaty wykrywać na save (HashSet + warning) - bez tego
utrata danych jest cicha.

## Trade-off
Sibling index jest stabilny tylko przy stabilnej hierarchii; obiekty tworzone w runtime
w niedeterministycznej kolejności nadal mogą mieć niestabilne klucze między sesjami.
GUID to jedyne rozwiązanie w pełni odporne.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260622-1412-saveload-order-event-doublecount|Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu]] - wspolne: isaveable, save-system
- [[20260702-2200-save-system-missing-key-reset|Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)]] - wspolne: isaveable, save-system
- [[20260712-1820-save-migration-schema-version-gate|Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu]] - wspolne: isaveable, save-system
- [[awake-init-for-isaveable-with-dependencies|Awake-Init for ISaveable with Dependencies]] - wspolne: isaveable, save-system
- [[isaveable-contract|ISaveable Contract]] - wspolne: isaveable, save-system
<!-- /POWIAZANE:auto -->
