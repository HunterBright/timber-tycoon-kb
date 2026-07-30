---
title: Unity CLI + com.unity.pipeline jako most agenta do żywego edytora
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-28'
project: Kerf - Sawmill Tycoon
tags:
- unity
- cli
- mcp
- agent-tooling
- automation
- build-verification
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Unity CLI + com.unity.pipeline jako most agenta do żywego edytora

## When to use
Kiedy agent (Claude Code, Codex, Copilot) ma sterować Unity: odczytywać stan sceny,
mierzyć wydajność, uruchamiać testy/build, wymuszać kompilację. Zastępuje wolne
mosty firm trzecich (Coplay, mosty społecznościowe) oficjalnym narzędziem Unity.
Wymaga Unity 6.0+.

## Steps
1. Instalacja CLI (Windows, kanał beta):
   `$env:UNITY_CLI_CHANNEL='beta'; irm https://public-cdn.cloud.unity3d.com/hub/prod/cli/install.ps1 | iex`
   Binarka ląduje w `%LOCALAPPDATA%\Unity\bin\unity.exe`, PATH użytkownika jest dopisywany.
2. `unity pipeline install --project-path <proj>` dokłada `com.unity.pipeline` do manifest.json.
3. Kliknąć w okno edytora (albo wymusić focus programowo) - Unity przelicza paczki
   dopiero po odzyskaniu focusu.
4. Poll `unity pipeline list` aż kolumna `Server Reachable` = true (u nas ~2 min, port 7800).
5. `unity mcp configure claude-code` dopisuje wpis MCP do konfiguracji agenta.
6. `unity list` = spis poleceń, `unity command <name> --json` = wykonanie na żywym edytorze.

## Why this works
Paczka pipeline wystawia lokalne API HTTP w procesie edytora, a CLI jest jego klientem.
Nie ma pośrednika w chmurze ani serwera firmy trzeciej, więc opóźnienie to czysty
lokalny round-trip. Pomiar: 138 poleceń, odpowiedź 1,3 s. Ten sam odczyt stanu edytora
przez Coplay MCP: ~6 minut.

## Trade-offs
- CLI i paczka są w wersji beta/eksperymentalnej (CLI 1.0.0-beta.3, paczka 0.4.0-exp.1).
- Paczka wchodzi do manifest.json projektu, czyli do repozytorium zespołu.
- Stary Unity MCP (relay) pozostaje wspierany, więc migracja nie jest wymuszona.

## Variants
- `unity test` / `unity build` działają bez otwartego edytora (spawnują batchmode).
- `get_performance_stats` daje draw calls / trójkąty / czas klatki jedną komendą -
  tania automatyczna kontrola budżetu wydajności zamiast oglądania Statistics ręcznie.

## GOTCHA
- Wpis MCP tworzony przez `unity mcp configure` używa gołej nazwy `unity`. Sesje agenta
  uruchomione PRZED instalacją mają stary PATH i most im nie wstanie. Wpisać pełną ścieżkę.
- Restart potrzebny tylko w tej sesji/karcie agenta, która ma dostać nowy most. Pozostałe
  sesje działają dalej na swoim komplecie i dostaną go przy własnym starcie.
- `unity pipeline install` działa bez logowania; `unity auth login` wymaga interakcji
  w przeglądarce i potrafi wygasnąć, jeśli nikt nie kliknie.
