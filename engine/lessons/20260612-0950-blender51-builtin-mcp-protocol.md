---
title: Blender 5.1 ma wbudowane rozszerzenie MCP - niekompatybilne ze starym blender-mcp
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-12'
project: Kerf - Sawmill Tycoon
tags:
- blender
- mcp
- blender-5.1
- socket
- protocol
- bridge
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Blender 5.1 ma wbudowane rozszerzenie MCP - niekompatybilne ze starym blender-mcp

## Problem
Po aktualizacji do Blendera 5.1 stary mostek MCP (addon ahujasid/blender-mcp + serwer `uvx blender-mcp`) przestał działać z objawem: **połączenie TCP na port 9876 jest przyjmowane, ale Blender nigdy nie odpowiada** ("Incomplete JSON response received"). Restart Blendera i reconnect addona nic nie dają.

## Root cause
Blender 5.1 dostarcza **oficjalne rozszerzenie MCP** (`extensions/user_default/mcp/mcp_to_blender_server.py`), które zajmuje ten sam port **9876**, ale używa innego protokołu:
- żądania/odpowiedzi to JSON rozdzielany **bajtem null** (`\0`), nie goły JSON
- format żądania: `{"type": "execute", "code": "<python>", "strict_json": <bool>}`
- kod musi ustawić zmienną `result` (dict); stdout/stderr są przechwytywane i zwracane
- odpowiedź: `{"status": "ok"|"error", "result": {...}, "stdout": "..."}` + `\0`

Stary serwer MCP wysyła goły JSON `{"type":"execute_code",...}` bez terminatora → wbudowany serwer czeka w nieskończoność na `\0` → cisza.

## Fix / workaround
Ominąć starego pośrednika i mówić bezpośrednio do wbudowanego serwera przez surowy socket TCP z ramkami null-terminated. Działający helper PowerShell: `_TempEditor/blender_exec.ps1` w repo Timber_Tycoon (wysyła plik .py, czeka na odpowiedź do `\0`).

## Diagnostyka na przyszłość
1. `Get-NetTCPConnection -LocalPort 9876` - kto słucha (PID Blendera = server żyje)
2. Surowy socket test: wyślij JSON bez `\0` - brak odpowiedzi = wbudowany serwer 5.1; odpowiedź = stary addon
3. Źródło protokołu: `%APPDATA%\Blender Foundation\Blender\5.1\extensions\user_default\mcp\mcp_to_blender_server.py`

## Konsekwencja
Narzędzia `mcp__blender-mcp__*` w sesji Claude Code są martwe przy Blenderze 5.1+, dopóki serwer MCP po stronie klienta nie zostanie zaktualizowany do nowego protokołu. Viewport screenshot przez stare narzędzie też nie działa - zamiast tego render do PNG przez `bpy.ops.render.render(write_still=True)` i odczyt pliku.
