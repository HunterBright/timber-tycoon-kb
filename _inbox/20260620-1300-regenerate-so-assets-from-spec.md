---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, scriptableobject, editor-scripting, assetdatabase, data-pipeline, codegen]
date: 2026-06-20
status: draft
---

# Regeneruj assety ScriptableObject z kodowego "spec" zamiast ręcznie edytować YAML/GUID

## Problem
Ręczna edycja plików `.asset` (YAML) Unity jest podatna na błędy, a **utworzenie NOWEGO assetu** wymaga wygenerowania unikalnego GUID-a w `.meta` i ręcznego wpięcia referencji w assecie-kontenerze (np. liście). To kruche i łatwo rozjechać dane.

## Wzorzec
Trzymaj **jedno źródło prawdy w kodzie** — statyczny builder zwracający instancje SO w pamięci (np. `public static QuestLineData BuildLine()` tworzące `ScriptableObject.CreateInstance<...>()`). Następnie mały skrypt edytorowy regeneruje pliki `.asset` z tego buildera:

```csharp
var line = SomeSpec.Build();                 // temp instancje SO w pamięci
for (int i = 0; i < line.items.Count; i++)
{
    var tmp = line.items[i];
    string path = dir + "Item_" + tmp.id + ".asset";
    var existing = AssetDatabase.LoadAssetAtPath<ItemSO>(path);
    if (existing != null)
    {
        EditorUtility.CopySerialized(tmp, existing); // nadpisuje treść, ZACHOWUJE GUID
        EditorUtility.SetDirty(existing);
        line.items[i] = existing;                    // kontener ma wskazywać PERSISTED asset
    }
    else
    {
        AssetDatabase.CreateAsset(tmp, path);        // nowy asset = nowy GUID, automatycznie
    }
}
// asset-kontener (lista referencji) przebuduj na persisted assety:
var existingLine = AssetDatabase.LoadAssetAtPath<LineSO>(linePath);
if (existingLine != null) { existingLine.items = line.items; EditorUtility.SetDirty(existingLine); }
else AssetDatabase.CreateAsset(line, linePath);
AssetDatabase.SaveAssets();
AssetDatabase.Refresh();
```

## Dlaczego to działa
- **Jedno źródło prawdy** (kod), z którego deterministycznie powstają assety.
- `CopySerialized` na istniejącym assecie **zachowuje GUID** → referencje z innych miejsc nie pękają.
- `CreateAsset` dla nowych nadaje GUID **automatycznie** — zero ręcznego pisania `.meta`.
- Asset-kontener przebudowany w tym samym przebiegu → referencje zawsze spójne.
- Designer dalej może edytować `.asset` w Inspektorze; gdy trzeba, re-sync z kodu jednym uruchomieniem.

## Gotchas
- **Edytuj spec, potem regeneruj.** Nie edytuj `.asset` i spec osobno — rozjadą się (brak auto-synchronizacji). Jeśli zespół edytuje też w Inspektorze, ustal kto jest kanoniczny.
- Tylko zmienione assety pokażą się w gicie (CopySerialized identycznej treści = brak zmiany bajtów) — to dobrze, mały diff.
- Uruchamiaj regenerator przez menu item (`[MenuItem]`) lub headless (np. komenda edytorowa/MCP), nie z gry.
- Jeśli kontener jest ładowany przez `Resources.Load("path")`, GUID kontenera jest nieistotny (ładowanie po ścieżce) — ważne tylko, by istniał w `Resources/` i miał poprawne referencje.

## Walidacja
Linia questów Timber Tycoon (13 `ScriptableObject` + kontener `QuestLineData`) zregenerowana czysto jednym skryptem edytorowym: zachowane GUID-y istniejących, nowy `Quest_ReachReputation2.asset` utworzony z poprawnym GUID-em, lista kontenera przebudowana, zweryfikowane przez `Resources.Load` (13 questów, zero null-referencji).
