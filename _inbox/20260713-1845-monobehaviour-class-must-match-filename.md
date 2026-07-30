---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, monobehaviour, monoscript, serialization, missing-script, scene-corruption, build-crash]
severity: critical
time_lost: "~2h (crash buildu -> sekcja zwlok obiektu -> odtworzenie)"
date: 2026-07-13
status: draft
applies_to: [unity]
---

# Klasa MonoBehaviour dołożona przez AddComponent MUSI leżeć w pliku o swojej nazwie — inaczej scena psuje build

## Anty-wzorzec

Dwie klasy `MonoBehaviour` w jednym pliku, a ta „druga" jest dokładana do obiektu przez `AddComponent<T>()`
(np. ze skryptu setupowego w edytorze):

```csharp
// LacquerStand.cs
public class LacquerStand : MonoBehaviour { ... }          // zgodna z nazwa pliku

public class LacquerStandPoint : Interactable { ... }      // <-- BOMBA
```

```csharp
// SetupWorkbench.cs (edytorowy)
standPoint.AddComponent<LacquerStandPoint>();              // "dziala"
```

Wygląda niewinnie. C# na to pozwala, kompiluje się, `AddComponent` zwraca żywy komponent i **wszystko
działa w tej sesji Edytora**.

## Dlaczego to nie działa

Unity tworzy **jeden asset `MonoScript` na plik `.cs`**, i reprezentuje on **tylko** klasę o nazwie
zgodnej z nazwą pliku. Klasa „druga" w pliku **nie ma żadnego MonoScript**.

Kiedy Unity serializuje komponent do sceny/prefabu, zapisuje odwołanie `m_Script` → MonoScript.
Dla klasy bez MonoScript **nie ma czego zapisać** → w scenie zostaje wpis, którego silnik nie umie rozwiązać.

## Objawy (dwa, i drugi jest zabójczy)

1. **W Edytorze** (po ponownym otwarciu sceny / przeładowaniu domeny): komponent **nie wczytuje się wcale**.
   `GetComponents<Component>()` zwraca w tym miejscu `null`. Funkcja, którą pełnił, jest **cicho martwa** —
   u nas cała interakcja `E` na stanowisku (gracz nie mógł odebrać przedmiotu, i nikt tego nie powiązał
   z „drugą klasą w pliku").

2. **W buildzie**: deserializacja sceny **wywraca się** na tym wpisie:
   ```
   The referenced script on this Behaviour (Game Object '<null>') is missing!
   The file '.../level0' is corrupted! Remove it and launch unity again!
   [Position out of bounds!]
   Crash!!!
   ```
   **Natywny crash w 2 sekundy od startu**, zanim odpali się jakikolwiek własny kod. Gra nie daje się
   uruchomić w ogóle.

## To NIE jest zwykły „missing script" — standardowe narzędzia go nie ruszą

To najbardziej podchwytliwa część. Diagnostyka daje sprzeczne odpowiedzi:

| Sprawdzenie | Wynik |
|---|---|
| `GetComponents<Component>()` | zawiera `null` ✔ (widzi problem) |
| `GameObjectUtility.GetMonoBehavioursWithMissingScriptCount(go)` | **0** ✘ |
| `GameObjectUtility.RemoveMonoBehavioursWithMissingScript(go)` | **nic nie usuwa** ✘ |
| `PrefabUtility.IsPartOfPrefabInstance(go)` | `false` (to nie wina prefabu) |
| kasowanie pustych wpisów z `m_Component` przez `SerializedObject` | **nie działa** (Unity blokuje edycję tej tablicy) |

Sekcja zwłok obiektu (`SerializedObject` → `m_Component`) pokazuje sedno:

```
[0] ObjectReference = UnityEngine.Transform
[1] ObjectReference = UnityEngine.BoxCollider
[2] ObjectReference = NULL (entityId=568105584918856740) -> nie da sie rozwiazac
```

Wpis **nie jest pusty** — wskazuje na obiekt, którego **typu silnik nie potrafi rozwiązać**. Dlatego
`GetMonoBehavioursWithMissingScriptCount` (liczy MonoBehaviour z pustym `m_Script`) go nie widzi.

## Naprawa

1. **Przenieś klasę do pliku o jej nazwie** (`LacquerStandPoint.cs`). Dopiero wtedy Unity ma dla niej
   MonoScript i potrafi ją zapisać.
2. **Odtwórz obiekt od zera.** Uszkodzonego wpisu nie da się usunąć żadnym API — trzeba skasować
   GameObject i zbudować go ponownie (najlepiej 1:1 wg parametrów ze skryptu, który go stworzył:
   pozycja lokalna, warstwa, collider, komponenty).
3. **Skanuj scenę I prefaby** przed każdym buildem — `GetComponents<Component>()` szukające `null`
   (nie polegaj na `GetMonoBehavioursWithMissingScriptCount`, bo ten przegapi właśnie ten przypadek).

## Zapobieganie

- **Jedna klasa MonoBehaviour = jeden plik o jej nazwie.** Bez wyjątków dla „cienkich" klas pomocniczych.
- Klasy nie-MonoBehaviour (structs, enumy, klasy danych) mogą dzielić plik — problem dotyczy **tylko**
  typów dokładanych do GameObjectów.
- Jeśli setup sceny robi `AddComponent<T>()` ze skryptu edytorowego, **otwórz scenę ponownie i sprawdź,
  czy komponent przeżył** — w tej samej sesji będzie wyglądał na działający.

## Transferability

Czysty mechanizm Unity, niezależny od gatunku gry i wersji (potwierdzone na 6000.5.1f1). Ryzyko rośnie
w projektach, gdzie scena jest budowana **skryptami edytorowymi** zamiast ręcznie — tam `AddComponent`
„działa", nikt nie klika komponentu w Inspektorze i błąd wychodzi dopiero przy buildzie.

Reguła nadrzędna: **jeśli scena jest binarna, jej uszkodzenia nie widać w code review — widać je dopiero
w buildzie.** Tym bardziej trzeba budować wcześnie.

## Related
- [[runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it]]
- [[build-early-never-built-project-hides-editor-only-bugs]]
