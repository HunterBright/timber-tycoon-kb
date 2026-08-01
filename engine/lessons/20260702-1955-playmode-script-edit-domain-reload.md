---
title: Edycja skryptów w trakcie Play Mode zabija statyczne rejestry (domain reload w locie)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- play-mode
- domain-reload
- service-locator
- editor-workflow
- mcp
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Edycja skryptów w trakcie Play Mode zabija statyczne rejestry (domain reload w locie)

## Problem
Podczas długiej sesji Play Mode (test przez MCP czekający ~100 s) zedytowano pliki .cs.
Unity zrekompilowało i zrobiło domain reload W TRAKCIE gry: obiekty sceny przeżyły,
ale wszystkie STATYKI wyzerowane - rejestr ServiceLocatora pusty, managery tworzone
przez `[RuntimeInitializeOnLoadMethod]` nie odtworzyły się (atrybut odpala się raz,
po załadowaniu sceny, nie po mid-play reload). Sesja wyglądała na żywą, ale
`Services.Get<T>()` zwracało null - wyniki sond bezużyteczne.

## Zasada
Podczas aktywnej sesji Play Mode (zwłaszcza automatyzowanej przez MCP/skrypty):
NAJPIERW stop, POTEM edycje kodu. Jeśli sesja ma trwać w tle, nie dotykać .cs.

## Jak rozpoznać
Nagłe "nie znaleziono serwisu X" w logach mimo wcześniej działających systemów;
`Application.isPlaying` nadal true; kompilacja czysta.

## Transfer
Dotyczy każdego projektu Unity z ServiceLocatorem/statycznymi singletonami i workflow,
w którym agent/skrypt edytuje kod, gdy edytor jest w Play Mode (domyślne ustawienie
Script Changes While Playing = Recompile And Continue Playing).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260611-2055-editor-playmode-test-harness-quirks|Editor-driven Play Mode test automation - three engine quirks that break naive harnesses]] - wspolne: domain-reload, play-mode
- [[20260710-2250-unity-autonomous-smoke-runner-flag-file|Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników]] - wspolne: play-mode, mcp
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: play-mode, mcp
<!-- /POWIAZANE:auto -->
