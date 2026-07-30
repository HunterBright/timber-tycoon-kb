---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, editor-scripts, scene, prefab, idempotency, destructive, colliders]
date: 2026-06-24
status: draft
---

# Edytorowe skrypty „Setup X" które DestroyImmediate + Instantiate + ustawiają pozycję = niszczące — nie używaj ich do drobnych poprawek

## Co próbowaliśmy
Chcieliśmy tylko docisnąć rozmiar trigger-colliderów (stref klikania) na regałach. Zmiana
wylądowała wewnątrz istniejącego menu `Setup Storage Racks`, a użytkownik został poproszony o jego
uruchomienie. Skutek: regały **przeskoczyły na inne pozycje**.

## Dlaczego to nie działa
Skrypty typu „Setup X" w tym projekcie (np. `SetupStorageRacks`, `SetupVehicleColliders` częściowo)
robią pełną ODBUDOWĘ obiektu:
`GameObject.Find` → `DestroyImmediate(old)` → `InstantiatePrefab` → `transform.position = STAŁA` →
re-add komponentów. Pozycje są **zakodowane na sztywno** w skrypcie. Jeśli ktoś później ręcznie
przesunął obiekty w scenie (u nas „rack realign"), ponowne uruchomienie skryptu **kasuje to** i
wraca do starych stałych. To NIE jest idempotentne względem ręcznych zmian sceny.

## Reguła
Do INKREMENTALNYCH zmian (rozmiar collidera, materiał, jedna właściwość) NIGDY nie uruchamiaj
skryptu, który DestroyImmediate'uje i ponownie instancjonuje obiekt. Zamiast tego napisz osobne,
NIENISZCZĄCE menu, które znajduje ISTNIEJĄCE obiekty i zmienia tylko docelową właściwość w miejscu
(zero destroy/instantiate, zero `transform.position = ...`). U nas: `Fit Interaction Zones (in place)`
— znajduje `StorageRack`/`Car1/Cabin`, zmienia tylko `BoxCollider.size/center`, nie rusza pozycji.

## Diagnoza/odzysk (przydatne)
- `git status --short -- <scena>` PUSTE przy scenie LFS = użytkownik NIE zapisał → wystarczy
  przeładować scenę bez zapisu (dysk = poprawny stan). To najszybszy, bezstratny odzysk.
- Pozycje w żywym edytorze czytaj przez MCP (Coplay `get_game_object_info` → Bounds/Transform);
  parsowanie .unity YAML zawodzi dla instancji prefabów (nazwa/pozycja w `m_Modifications`, nie jako
  zwykłe `m_Name:`/`m_LocalPosition:`).

## Sygnał ostrzegawczy
Jeśli „drobna poprawka" wymaga uruchomienia skryptu z nazwą `Setup*`/`Generate*`/`Rebuild*` —
zatrzymaj się: prawdopodobnie odbuduje obiekty i zgubi ręczne ustawienia. Zrób osobny mutator in-place.
