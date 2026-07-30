---
type: lesson
project: Timber Tycoon
suggested-category: workflow/lessons
tags: [unity, build, ci, verification, editor-only-api, shader-find, technical-debt]
severity: high
time_lost: "3 bugi blokujace, wszystkie wyszly w ciagu 1h pierwszego buildu"
date: 2026-07-13
status: draft
applies_to: [unity]
---

# Projekt, który nigdy nie był budowany, hoduje całą klasę uśpionych błędów

## Problem

Projekt rozwijany ~rok wyłącznie w Edytorze (setki commitów, pełna rozgrywka, playtesty).
**Nigdy nie zrobiono buildu.** Pierwsza próba wyprodukowania `.exe` odsłoniła — w ciągu godziny —
**trzy niezależne błędy blokujące**, z których każdy sam w sobie uniemożliwiał wydanie gry:

1. **Player się nie kompilował.** `AssetDatabase.LoadAssetAtPath(...)` w kodzie runtime bez
   `#if UNITY_EDITOR` → `CS0103`. Analogiczne wywołanie 50 linii niżej w tym samym pliku było
   zabezpieczone — o tym po prostu zapomniano. Edytor kompiluje to bez mrugnięcia.
2. **Build padał przy starcie** (`level0 is corrupted!` → natywny crash). Jeden komponent w scenie
   miał nierozwiązywalne odwołanie do skryptu. Edytor to toleruje.
3. **Kolizje tworzone w runtime nie powstawały** (siatki bez Read/Write Enabled). Edytor działa
   zawsze — bug był całkowicie niewidoczny.

Wspólny mianownik: **każdy z nich „działa w Edytorze".** Żaden nie zostałby znaleziony przez code review,
testy jednostkowe ani playtesty w Edytorze.

## Root cause

Edytor Unity to **inne środowisko wykonawcze** niż player, i jest **bardziej wyrozumiały** w co najmniej
czterech wymiarach:

| Rzecz | Edytor | Build |
|---|---|---|
| Dane siatki z `isReadable: 0` | dostępne (AssetDatabase trzyma kopię) | **skasowane z RAM** |
| Komponent z nierozwiązywalnym skryptem | tolerowany, pokazany jako „Missing" | **wywraca deserializację sceny** |
| `UnityEditor.*` w kodzie runtime | kompiluje się | **nie kompiluje się** |
| `Shader.Find("X")` dla shadera bez materiału | znajduje (wszystko jest w AssetDatabase) | **zwraca null** (shader wycięty z buildu) |

Im dłużej projekt nie jest budowany, tym więcej takich rozbieżności się nawarstwia — i tym trudniej je
rozplątać, bo wychodzą **wszystkie naraz**.

## Solution

**Zrób build w pierwszym tygodniu projektu, nawet pustej sceny. Potem powtarzaj regularnie.**

Minimalna automatyzacja (bez CI, ~50 linii):

```csharp
[MenuItem("Projekt/Build/Windows x64")]
public static void BuildWindows()
{
    var report = BuildPipeline.BuildPlayer(new BuildPlayerOptions {
        scenes = EnabledScenes(),
        locationPathName = "Builds/Win64/Game.exe",
        target = BuildTarget.StandaloneWindows64,
        options = BuildOptions.Development   // pelne logi zamiast czarnej skrzynki
    });
    // wypisz report.steps[].messages typu Error do pliku
    if (Application.isBatchMode) EditorApplication.Exit(ok ? 0 : 1);
}
```

```powershell
# Unity MUSI byc zamkniete
Unity.exe -batchmode -quit -projectPath <proj> -executeMethod BuildTool.BuildWindows -logFile build.log
```

Dwa niuanse, które kosztują czas przy pierwszym podejściu:
- W trybie wsadowym Unity trzyma **pustą, nienazwaną scenę** — `EditorSceneManager.NewScene(..., Additive)`
  rzuci `Cannot create a new scene additively with an untitled scene unsaved`. Użyj `Single` w batchu,
  a `Additive` z otwartego Edytora (żeby nie ruszyć żywej sceny).
- `BuildPipeline.BuildPlayer` czyta sceny **z dysku**, nie z pamięci Edytora — niezapisane zmiany nie wejdą
  do buildu (co bywa i zaletą, i pułapką).

## Bonus: build jako narzędzie diagnostyczne

Sam build to za mało — build, który się **uruchamia i sam siebie sprawdza**, jest znacznie mocniejszy.
Wzorzec: **sonda bramkowana argumentem wiersza poleceń**, która przy starcie wykonuje krytyczne ścieżki kodu,
weryfikuje je **behawioralnie**, pisze raport obok `.exe` i kończy proces kodem 0/1.

```
Game.exe -meshaudit -logFile player.log
echo $LASTEXITCODE   # 0 = OK, 1 = cos padlo
```

Daje maszynowo sprawdzalny werdykt **z prawdziwego builda, bez grania** — i nadaje się wprost do CI.

## What didn't work

Próba **udowodnienia buildowych błędów bez buildu**. Dla siatek istnieje pozorna droga na skróty
(`Mesh.UploadMeshData(true)` — oficjalny sposób zwolnienia kopii siatki z pamięci procesora), ale
**w Edytorze to nic nie robi**: dane zostają, collider cookuje się normalnie. Szczegóły →
[[runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it]].

Wniosek ogólny: **jeśli klasa błędów definiuje się przez różnicę Edytor↔build, to Edytor z definicji
nie może być jej dowodem.** Nie szukaj obejść — zbuduj.

## Transferability

Każdy silnik z „trybem edytora" bogatszym niż runtime (Unity, Unreal, Godot w mniejszym stopniu).
Im więcej narzędzi edytorowych ma projekt (skrypty setupu sceny, generatory assetów), tym większe ryzyko,
że kod edytorowy przecieknie do runtime'u lub że scena zostanie zbudowana w stanie, którego player nie wczyta.

Reguła kciuka: **„działa u mnie w Edytorze" nie jest twierdzeniem o grze — to twierdzenie o Edytorze.**

## Related
- [[runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it]]
- [[monobehaviour-class-must-match-filename]]
