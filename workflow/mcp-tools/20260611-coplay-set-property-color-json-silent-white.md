---
title: 'Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-11'
project: Kerf - Sawmill Tycoon
tags:
- coplay
- mcp
- set_property
- color
- unity
- silent-failure
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white

## Problem
Setting a `Color` field via Coplay MCP `set_property` with a JSON-style value (e.g. `{"r":0.2,"g":0.4,"b":0.1,"a":1}`) **reports success** but actually writes **white** (1,1,1,1). No error, no warning - the only symptom is the object rendering white later.

## Fix
Pass the value as a plain comma-separated string: `0.2,0.4,0.1,1` (r,g,b,a). This parses correctly and writes the intended color.

## Detection rule
Any Coplay set_property call on a Color that "succeeded" but the object renders white → re-send with the comma format and re-verify visually. Treat every reported success on Color fields as unverified until seen rendered.

## Transferability
Any project driving Unity through Coplay MCP. General family: MCP tools that report success while silently coercing/defaulting an unparseable value - verify visually, not by tool status (see [[debugging-search-first-trust-render-check-upstream]] Rule 2).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] - wspolne: coplay, mcp
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: coplay, mcp
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: coplay, mcp
- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: coplay, mcp
- [[20260710-2252-coplay-execute-script-tmpro-compile-fail|Coplay execute_script nie kompiluje plików z `using TMPro;`]] - wspolne: coplay, mcp
- [[20260608-1503-mcp-scene-capture-omits-gizmos|MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
