---
title: Scena zostaje BINARNA mimo Force Text, a dwa oczywiste lekarstwa nie działają
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- serialization
- scene
- yaml
- force-text
- assetdatabase
- git-lfs
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Scena zostaje BINARNA mimo Force Text, a dwa oczywiste lekarstwa nie działają

## Objaw

`Editor Settings > Asset Serialization > Mode = Force Text`, potwierdzone dwoma niezależnymi
źródłami (`ProjectSettings/EditorSettings.asset` ma `m_SerializationMode: 2`, a runtime'owe
`EditorSettings.serializationMode` zwraca `ForceText`). Mimo to plik sceny jest binarny:
zaczyna się bajtami zerowymi i sygnaturą wersji edytora, nie od `%YAML 1.1`.

Wszystkie pozostałe assety w projekcie (`.mat`, `.prefab`, `.asset`) są prawidłowo tekstowe.
Binarne są **wyłącznie sceny** (`.unity`), i to wszystkie, także stare kopie zapasowe.

## Co NIE działa (zmierzone, nie zgadywane)

**1. `AssetDatabase.ForceReserializeAssets(new[]{ scenePath })` jest ciche i bezskuteczne.**
Nie rzuca wyjątku, nie loguje ostrzeżenia, zwraca sterowanie normalnie. Plik po operacji ma
identyczną sumę SHA256 co przed. Powód: `SceneAsset` jest tylko uchwytem do pliku, a nie
grafem obiektów sceny, więc AssetDatabase nie ma czego wczytać i przepisać. Dla materiału,
prefabu czy ScriptableObjecta ta sama komenda działa poprawnie.

**2. `EditorSceneManager.SaveScene` też nie konwertuje.** Sprawdzone w czterech wariantach,
każdy zwrócił `true` i każdy dał plik binarny bajt w bajt identyczny ze źródłem:
- `SaveScene(scene, nowaSciezka, saveAsCopy: true)`
- `SaveScene(scene, tasamaSciezka)` po `MarkSceneDirty`
- `SaveScene(scene, tasamaSciezka)` po `EditorUtility.SetDirty` na **wszystkich** 7913 obiektach
  i 21937 komponentach
- `SaveScene(scene, nowaSciezka)` z `saveAsCopy` pominiętym (czyli `false`)

Czas zapisu pliku się zmienia, więc Unity naprawdę pisze. Pisze binarnie.

## Test rozstrzygający

Nowa, pusta scena utworzona przez `EditorSceneManager.NewScene` i zapisana w tym samym
projekcie, tym samym uruchomieniem edytora, wychodzi **tekstowa** (`%YAML 1.1`, 4398 B).

Wniosek: ustawienie projektu działa poprawnie. Format nie jest brany z ustawień przy
każdym zapisie, tylko **wędruje razem ze sceną wczytaną z pliku**. Scena raz zapisana
binarnie zostaje binarna przy każdym kolejnym zapisie, niezależnie od ścieżki docelowej,
flagi kopii i wymuszonego oznaczenia zmian.

## Co z tego wynika

Jedyny udokumentowany mechanizm Unity, który przestawia format **istniejących** plików, to
zmiana `Asset Serialization Mode` w Editor Settings. Ta zmiana z definicji dotyka całego
projektu, więc nie da się nią przekonwertować jednej sceny.

Wariant najmniej inwazyjny, jeśli mass-reserializacja jest dopuszczalna: przełączyć tryb na
`Mixed`, zatwierdzić, a potem z powrotem na `Force Text`. Trasa przez `Force Binary` oznacza
dwa pełne przebiegi i po drodze zamienia cały projekt na binarny. Przy trasie przez `Mixed`
assety już tekstowe zostaną przepisane bajt w bajt (git ich nie zobaczy), a realnie zmienią
się tylko sceny.

## Pułapka towarzysząca: LFS zjada całą korzyść

Jeśli `.gitattributes` ma `*.unity filter=lfs … -text`, to nawet po udanej konwersji
`git diff` pokaże wyłącznie zmianę wskaźnika LFS, a nie treść sceny. Czyli główny powód
przechodzenia na YAML (czytelne różnice w gicie) nie zostaje osiągnięty. Konwersję formatu
i wyjęcie scen z LFS trzeba traktować jako jedną decyzję, nie dwie osobne.

## Jak diagnozować szybko

1. Sprawdź nagłówek pliku, nie ustawienie: pierwsze bajty `.unity`.
2. Porównaj z innym typem assetu (`.mat`) - jeśli tamten jest tekstowy, ustawienie działa.
3. Zapisz nową pustą scenę do pliku testowego. To rozdziela „projekt jest źle ustawiony"
   od „ten konkretny plik jest zapieczony w binarnym".
4. Sumy SHA256 przed i po operacji, zawsze. Bez nich `SaveScene` zwracające `true`
   wygląda jak sukces.
