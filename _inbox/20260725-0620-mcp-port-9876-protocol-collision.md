---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: workflow/lessons
tags: [mcp, blender, tooling, sockets, diagnosis]
severity: high
time_lost: "~40 min"
date: 2026-07-25
status: draft
applies_to: [blender-5.x, blender-mcp, claude-code]
---

# Dwa serwery MCP na porcie 9876: "Incomplete JSON response" to konflikt protokolu, nie zawieszony Blender

## Problem
Wszystkie narzedzia `mcp__blender-mcp__*` w Claude Code zwracaly
`Communication error with Blender: Incomplete JSON response received`, a wywolania wisialy
po ~40 s i wpadaly w tlo. Blender byl uruchomiony, port 9876 nasluchiwal, proces byl
`Responding: True` i zjadal 0% CPU. Wygladalo na zamrozony glowny watek Blendera
(hipoteza: `bpy.app.timers` nie odpalaja, gdy petla zdarzen spi).

## Root cause
Na porcie 9876 nasluchiwal INNY serwer, niz ten, do ktorego mowi klient:

- Blender 5.1 ma wlasna, oficjalna wtyczke `MCP` (Blender Lab, `extensions/user_default/mcp`),
  ktora tez domyslnie bierze port 9876, ale wymaga ramek JSON **zakonczonych bajtem `\0`**
  (`mcp_to_blender_server.py`: `if b"\0" not in client.buffer: ...`).
- Spolecznosciowy `blender-mcp` (ahujasid) wysyla czysty JSON **bez terminatora**
  i tez celuje w 9876 (`DEFAULT_PORT = 9876`).

Efekt: oficjalna wtyczka odbiera bajty, wiecznie czeka na `\0`, po ~40 s odsyla
`{"status": "error", "message": "Client timed out"}\x00`. Klient nie umie tego zjesc
(trailing NUL + inny schemat) i raportuje "Incomplete JSON response received".
Wnioskowanie "Blender zawieszony / timery nie odpalaja" bylo calkowicie bledne.

## Solution
Diagnoza (30 sekund, zamiast zgadywania): surowy test gniazda z pominieciem MCP.
Polacz sie na port, wyslij komende, wypisz DOKLADNE bajty odpowiedzi i czas.
Trailing `\x00` w odpowiedzi natychmiast identyfikuje serwer po drugiej stronie.

Rozwiazanie docelowe (koegzystencja, bez psucia niczego):
- oficjalna wtyczka zostaje na 9876 (jesli cos juz z niej korzysta),
- spolecznosciowa wtyczka dostaje inny port w panelu (`blendermcp_port`),
- klient MCP dostaje `BLENDER_PORT=<nowy>` w `env` swojego wpisu w konfiguracji MCP
  (`blender_mcp/server.py` czyta `BLENDER_HOST` / `BLENDER_PORT`).

## What didn't work
- Sprawdzanie `Responding` / CPU procesu Blendera: mowi tylko, ze okno pompuje komunikaty,
  nic o tym, czy wtyczka odpowiada.
- Budzenie petli zdarzen Blendera przez `PostMessage(WM_MOUSEMOVE)` + `RedrawWindow`:
  CPU drgnelo, odpowiedzi to nie zmienilo (bo problem byl gdzie indziej).
- Czytanie kodu wtyczki, o ktorej ZALOZYLEM, ze nasluchuje. Najpierw ustal KTO trzyma port
  (`Get-NetTCPConnection` -> `OwningProcess`), potem czytaj wlasciwy kod.

## Transferability
Kazdy projekt z lokalnym mostem "LLM -> aplikacja" po TCP: gdy dwie wtyczki/serwery
z tej samej rodziny biora ten sam domyslny port, objaw wyglada jak zawieszenie aplikacji,
a jest rozjazdem protokolu (ramkowanie: NUL-terminated vs newline vs czysty JSON).
Zasada ogolna: **najpierw kto trzyma port, potem surowe bajty, na koncu hipotezy o watkach.**

## Related
- [[blender51-mcp-bridge]] (projektowy mostek `_TempEditor/blender_exec.ps1` uzywa wlasnie
  protokolu NUL-terminated na 9876)
