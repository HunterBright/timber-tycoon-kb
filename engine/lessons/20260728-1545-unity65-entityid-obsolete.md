---
title: 'Unity 6.5: instance ID w SerializedProperty to teraz EntityId, a rzutowanie na int też jest zakazane'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- unity-6.5
- editor-scripting
- serializedproperty
- entityid
- compile-error
- safe-mode
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Unity 6.5: instance ID w SerializedProperty to teraz EntityId, a rzutowanie na int też jest zakazane

## Objaw
Projekt startuje w SAFE MODE. W logu (`<projekt>/Logs/Editor.log`, NIE `%LOCALAPPDATA%\Unity\Editor\Editor.log`)
jeden błąd, powtórzony przy każdej próbie kompilacji:

    error CS0619: 'SerializedProperty.objectReferenceInstanceIDValue' is obsolete:
    Use objectReferenceEntityIdValue instead

CS0619 to obsolete oznaczone jako BŁĄD, nie ostrzeżenie - żadne ustawienie projektu tego nie łagodzi.

## Pułapka (pierwsza naprawa NIE działa)
Naturalna poprawka `int id = property.objectReferenceEntityIdValue;` kompiluje się w głowie,
bo `EntityId` ma niejawną konwersję na `int`. Ale ta konwersja jest **tak samo przestarzała**:

    error CS0619: 'EntityId.implicit operator int(EntityId)' is obsolete:
    EntityId will not be representable by an int in the future.

Czyli wymiana nazwy właściwości nie wystarcza - trzeba przestać traktować identyfikator jak liczbę.

## Poprawka
```csharp
EntityId referenceEntityId = property.objectReferenceEntityIdValue;
if (property.objectReferenceValue == null && !referenceEntityId.Equals(EntityId.None))
{
    string note = "brakująca referencja; ID=" + referenceEntityId.ToString();
}
```
`EntityId` (namespace `UnityEngine`) ma: `None` (właściwość statyczna), `IsValid()`, `Equals(EntityId)`,
`CompareTo`, `ToString()`, `ToULong`/`FromULong`. Sklejanie ze stringiem rób przez jawne `ToString()` -
bez tego kompilator może wybrać ścieżkę przez przestarzałą konwersję na `int`.

## Jak sprawdzić API bez internetu
Pełna lista członków typu leży w dokumentacji XML obok DLL-i edytora:
`<Unity>/Editor/Data/Managed/UnityEngine/UnityEngine.CoreModule.xml` - szukać `<member name="M:UnityEngine.EntityId`.
Szybsze i pewniejsze niż zgadywanie z pamięci.

## Powiązana pułapka: wyjście z Safe Mode bez klikania
Unity przelicza assety dopiero, gdy okno ODZYSKA focus. Samo `SetForegroundWindow` działa tylko wtedy,
gdy okno naprawdę było w tle; gdy już było aktywne, nic się nie dzieje i wygląda to jak "poprawka nie pomogła".
Pewny cykl: `ShowWindow(hWnd, 6)` (minimalizuj) → 2 s → `ShowWindow(hWnd, 9)` + `SetForegroundWindow`.
Po znikniętym błędzie Unity samo wychodzi z Safe Mode (tytuł okna traci "SAFE MODE").
