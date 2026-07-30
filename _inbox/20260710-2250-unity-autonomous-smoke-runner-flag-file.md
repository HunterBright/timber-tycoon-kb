---
type: pattern
project: Timber Tycoon
suggested-category: engine/patterns
tags: [unity, testing, smoke-tests, autonomous, mcp, coplay, play-mode]
date: 2026-07-10
status: draft
---

# Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników

## Problem
Sesja autonomiczna (Claude + MCP) musi uruchamiać i CZYTAĆ wyniki testów Play Mode bez człowieka.
Dwa zgryzy: (1) `get_unity_logs` (Coplay) milknie po wyjściu z Play Mode - logi konsoli są
nieosiągalne po fakcie; (2) test nie może wymagać ręcznego włączania obiektu w hierarchii.

## Wzorzec (zwalidowany na 6 przebiegach, suite 13 scenariuszy)
1. **Bootstrap przez plik-flagę**: statyczna metoda z `[RuntimeInitializeOnLoadMethod(AfterSceneLoad)]`
   w klasie testu sprawdza `File.Exists(<repo>/_Handoff/RUN_SMOKE.flag)`. Jeśli jest: czyta treść
   (= numer bramki, ile scenariuszy uruchomić), **kasuje flagę OD RAZU** (crash suite nie zapętli
   kolejnych Play), spawnuje GameObject z komponentem testu. Gate w treści flagi pozwala tej samej
   klasie rosnąć wraz z fazami budowy (scenariusze ponad bramkę logują SKIP).
2. **Wyniki do pliku, nie tylko do konsoli**: każda linia PASS/FAIL idzie równolegle
   `Debug.Log` + `File.AppendAllText(<repo>/_Handoff/SMOKE_RESULTS.txt)`. Plik truncowany na
   starcie suite; ostatnia linia `===== END (passed X/Y) =====` jest markerem końca.
   Częściowe wyniki przeżywają zawieszkę Unity.
3. **Pętla orkiestratora** (Claude): zapisz flagę -> `play_game` (MCP) -> polluj plik wyników
   do `===== END` (timeout ~240 s) -> `stop_game` -> parsuj -> bramkuj commit fazy.
4. **Seeding stanu przez publiczne API save**: zamiast grzebać w prywatnych polach managera,
   round-trip `JsonUtility.FromJson<XSaveData>(mgr.GetSaveData())` -> modyfikacja -> `mgr.LoadSaveData(...)`.
   Load replayuje efekty/eventy jak przy prawdziwym wczytaniu - test dostaje spójny stan za darmo.
5. **Autosave OFF na czas suite** (`SetAutoSaveEnabled(false)`, przywrócenie w finally/Finish) +
   testy save/load na NAJWYŻSZYM slocie z **backupem pliku gracza** (File.Copy przed, File.Move po)
   - sloty bywają ograniczone (tu 0-2), a test nie może zniszczyć save'a właściciela.

## Dlaczego to działa
Plik na dysku jest jedynym kanałem, który przeżywa granicę Play Mode/Edit Mode i jest czytelny
dla zewnętrznego orkiestratora bez API Unity. Kasowanie flagi przed startem = at-most-once.

## Anti-gotcha
Nie polegaj na `get_unity_logs` po `stop_game` - wyniki muszą być na dysku zanim Play się skończy.
