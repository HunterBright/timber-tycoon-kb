---
type: lesson
project: Timber_Tycoon
suggested-category: engine/lessons
tags: [unity, play-mode, domain-reload, service-locator, editor-workflow, mcp]
date: 2026-07-02
status: draft
severity: medium
---

# Edycja skryptów w trakcie Play Mode zabija statyczne rejestry (domain reload w locie)

## Problem
Podczas długiej sesji Play Mode (test przez MCP czekający ~100 s) zedytowano pliki .cs.
Unity zrekompilowało i zrobiło domain reload W TRAKCIE gry: obiekty sceny przeżyły,
ale wszystkie STATYKI wyzerowane — rejestr ServiceLocatora pusty, managery tworzone
przez `[RuntimeInitializeOnLoadMethod]` nie odtworzyły się (atrybut odpala się raz,
po załadowaniu sceny, nie po mid-play reload). Sesja wyglądała na żywą, ale
`Services.Get<T>()` zwracało null — wyniki sond bezużyteczne.

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
