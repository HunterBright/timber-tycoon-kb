---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, build, debug-keys, cheats, scripting-defines, release-safety]
date: 2026-07-13
status: draft
---

# Build z cheatami przez `extraScriptingDefines` — zamiast „odblokuj i pamiętaj cofnąć"

## Problem

Klawisz debugowy (skok do późnego stanu gry, dosypanie surowców, odblokowanie poziomu) jest zamknięty w `#if UNITY_EDITOR`, więc **w buildzie nie istnieje**. Ale playtester ocenia rzecz, do której w normalnej grze trzeba dojść przez długą pętlę rozgrywki — bez skrótu ogląda ją po kilkunastu minutach grania.

Odruchowe rozwiązanie: zdjąć `#if UNITY_EDITOR`, zbudować, **i zapamiętać, żeby założyć go z powrotem przed wydaniem**.

To jest mina. Notatki „pamiętaj cofnąć" gubią się, a cheat-klawisz w wydanej grze to nie kosmetyka.

## Wzorzec

Rozdziel **buildy**, a nie kod.

1. Zmień bramkę pliku na `#if UNITY_EDITOR || TWOJ_PROJEKT_CHEATS`.
2. Dodaj osobne wejście budujące, które podaje tę definicję **wyłącznie na czas tego jednego builda** — w Unity służy do tego `BuildPlayerOptions.extraScriptingDefines`. Kluczowe: **to NIE dotyka ustawień projektu** (Player Settings → Scripting Define Symbols zostają czyste), więc nie ma czego cofać.
3. Wypuść go do **osobnego folderu** (`Builds/Win64_Cheats/`), żeby nikt nie pomylił wersji do oglądania z grą.

```csharp
public static void BuildWithCheats()
{
    BuildGame(output: "Builds/Win64_Cheats/Game.exe",
              extraDefines: new[] { "TWOJ_PROJEKT_CHEATS" });
}

var opts = new BuildPlayerOptions
{
    // ...
    extraScriptingDefines = extraDefines   // null dla zwyklego builda
};
```

**Efekt: zwykły build NIE MA JAK zawierać cheatów — nawet gdyby ktoś zapomniał o wszystkim.** Bezpieczeństwo wynika z konstrukcji, nie z dyscypliny.

## Dwa warunki, o których łatwo zapomnieć

1. **Plik cheata nie może używać ŻADNEGO API edytora** (`UnityEditor`, `AssetDatabase`, `EditorUtility`...). Dopóki siedział w `#if UNITY_EDITOR`, wolno mu było. Gdy zaczyna się kompilować do gracza — jedno takie wywołanie i **player się nie skompiluje** (CS0103). Zgrepuj plik ZANIM zmienisz bramkę. W tym projekcie dokładnie tak padł kiedyś cały build.
2. **Zweryfikuj rozdział empirycznie, nie na słowo.** Jeśli klawisz loguje coś przy starcie (np. `[JUMP] aktywny`), uruchom oba buildy i policz wystąpienia w logu:

```powershell
# czysty build: oczekiwane 0 wystapien
# build z cheatem: oczekiwane >0
(Select-String -Path player_clean.log  -Pattern "\[JUMP\]").Count
(Select-String -Path player_cheats.log -Pattern "\[JUMP\]").Count
```

Bez tego „chyba się nie skompilował do gry" jest przypuszczeniem, a nie faktem.

## Powiązane

- [[editor-cannot-simulate-build]] — ten sam nawyk: dowodem jest zbudowany plik `.exe`, nie Edytor.
