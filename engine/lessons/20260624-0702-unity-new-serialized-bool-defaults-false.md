---
title: Nowe pole `bool` na istniejących assetach deserializuje się do `false`, nie do inicjalizatora C#
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-24'
project: Kerf - Sawmill Tycoon
tags:
- unity
- serialization
- scriptableobject
- backward-compat
- save-migration
- default-values
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Nowe pole `bool` na istniejących assetach deserializuje się do `false`, nie do inicjalizatora C#

## Problem / kontekst
Dodając nową flagę do klasy `[Serializable]` osadzonej w istniejącym assecie (np. `QuestObjective`
w liście wewnątrz `QuestData.asset`), naturalnym odruchem jest `public bool showArrow = true;` z
„bezpiecznym" domyślnym `true`. **Pułapka:** gdy Unity wczytuje STARY asset, w którego YAML tego pola
jeszcze nie ma, NIE stosuje inicjalizatora C# - wpisuje `default(T)`, czyli dla `bool` → `false`.

Efekt: wszystkie istniejące obiekty dostają `showArrow = false`, mimo inicjalizatora `= true`. W tym
projekcie oznaczałoby to zniknięcie strzałek-kompasu we WSZYSTKICH questach samouczka (regresja),
bo questy ładują się z assetu `Resources/Quests/TutorialQuestLine`, a tylko nowe questy
po-samouczkowe pochodzą z fallbacku `BuildLine()` w pamięci (tam inicjalizator C# działa).

## Reguła
Dla flag opcjonalnych dodawanych do ISTNIEJĄCYCH serializowanych assetów: **projektuj domyślną
wartość tak, by `default(T)` = pożądane zachowanie „nie zmieniaj nic"**. Dla boola znaczy to zwykle
odwrócenie sensu: zamiast `showX = true` (opt-out, łamie stare assety) użyj `hideX = false`
(opt-in, stare assety zachowują się jak dawniej). Wtedy brak pola w YAML → `false` → „pokaż" =
zachowanie sprzed zmiany.

## Jak rozpoznać, czy ryzyko występuje
- Pole jest `[SerializeField]`/public w `MonoBehaviour`/`ScriptableObject` LUB w `[Serializable]`
  klasie/strukcie osadzonej w takim assecie.
- Asset(y) z tym typem ISTNIEJĄ na dysku i są wczytywane (Resources/Addressables/scena), a nie
  budowane zawsze od zera w kodzie.
- Jeśli wszystko jest budowane w kodzie (jak fallback `BuildLine()`), inicjalizator zadziała - ryzyka brak.

## Alternatywy
- Dopisać pole do wszystkich assetów (skrypt/migracja) - pewne, ale pracochłonne i wymaga reimportu.
- Użyć `OnAfterDeserialize`/wersjonowania do uzupełnienia - cięższe.
- Najtańsze i najbezpieczniejsze: dobrać semantykę tak, by `default(T)` był poprawnym defaultem.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260712-1925-unity-so-asset-overrides-initializers|ScriptableObject .asset serializuje wartosci i WYGRYWA nad inicjalizatorami pol w C#]] - wspolne: serialization, scriptableobject
- [[20260722-1625-measure-before-fixing-serialization-hunch|Brakujący klucz w assecie NIE oznacza zera - zmierz, zanim "naprawisz"]] - wspolne: serialization, scriptableobject
<!-- /POWIAZANE:auto -->
