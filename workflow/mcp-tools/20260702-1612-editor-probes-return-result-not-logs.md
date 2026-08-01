---
title: 'Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- mcp
- coplay
- editor-scripting
- diagnostics
- play-mode
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log

## Kontekst
Autonomiczna praca w Unity przez MCP (Coplay `execute_script`): skrypty diagnostyczne („sondy") odpalane w Edit/Play Mode do sprawdzania stanu sceny, magazynu, kolejek NPC itd.

## Problem
Bufor logów konsoli czytany przez MCP (`get_unity_logs`) potrafi **zamarznąć po wyjściu z Play Mode** - kolejne `Debug.Log` z sond edytorowych nie pojawiają się w odczycie, mimo że skrypt wykonał się poprawnie (Success:true). Diagnoza w ciemno.

## Wzorzec
Metoda sondy zwraca `string` - MCP oddaje go bezpośrednio jako `Result` wywołania, z pominięciem konsoli:
```csharp
public class MyProbe
{
    public static string Execute()
    {
        var sb = new StringBuilder("[PROBE]\n");
        sb.AppendLine($"stan: {...}");
        return sb.ToString(); // trafia do Result - zawsze czytelne
    }
}
```
Dodatkowe zalety: raport atomowy (nie ginie między innymi logami), łatwe asercje „oczekiwane vs faktyczne" w jednej odpowiedzi, działa identycznie w Edit i Play Mode. `Debug.Log` zostawiać tylko jako ślad wtórny.

## Kiedy stosować
Każda sonda/diagnostyka odpalana przez MCP, szczególnie po przejściach Play↔Edit. Długie przebiegi w Play Mode (smoke testy) nadal mogą logować do konsoli - tam bufor działa; zamarza dopiero po wyjściu z Play.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260710-2250-unity-autonomous-smoke-runner-flag-file|Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników]] - wspolne: play-mode, coplay, mcp
- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: coplay, editor-scripting, mcp
- [[20260710-2252-coplay-execute-script-tmpro-compile-fail|Coplay execute_script nie kompiluje plików z `using TMPro;`]] - wspolne: coplay, editor-scripting, mcp
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: coplay, mcp
- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] - wspolne: coplay, mcp
- [[20260611-coplay-set-property-color-json-silent-white|Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
