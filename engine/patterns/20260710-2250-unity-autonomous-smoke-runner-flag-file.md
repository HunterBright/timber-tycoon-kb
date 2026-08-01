---
title: 'Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- testing
- smoke-test
- autonomous
- mcp
- coplay
- play-mode
applies_to: []
source: ''
promoted: '2026-07-30'
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

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: play-mode, coplay, mcp
- [[20260720-1308-pula-jednoelementowa-udaje-pelne-pokrycie|Test losujacy jeden element z puli o rozmiarze 1 udaje pelne pokrycie]] - wspolne: smoke-test, testing
- [[20260722-1652-relative-only-test-blind-to-common-mode-error|Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)]] - wspolne: smoke-test, testing
- [[20260611-2055-editor-playmode-test-harness-quirks|Editor-driven Play Mode test automation - three engine quirks that break naive harnesses]] - wspolne: play-mode, testing
- [[scriptableobject-playmode-persistence|ScriptableObject changes in Play Mode DO persist after exit]] - wspolne: play-mode, testing
- [[20260702-1955-playmode-script-edit-domain-reload|Edycja skryptów w trakcie Play Mode zabija statyczne rejestry (domain reload w locie)]] - wspolne: play-mode, mcp
<!-- /POWIAZANE:auto -->
