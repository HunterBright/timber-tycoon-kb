---
title: Coplay execute_script nie kompiluje plików z `using TMPro;`
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Kerf - Sawmill Tycoon
tags:
- unity
- coplay
- mcp
- execute_script
- tmpro
- roslyn
- editor-scripting
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Coplay execute_script nie kompiluje plików z `using TMPro;`

## Symptom
`execute_script` na pliku edytorowym zwraca mylący błąd:
`Could not find any resources appropriate for the specified culture ... CSharpResources.resources ... Coplay-v8.15.1`
(to zamaskowany komunikat diagnostyki Roslyna - realny błąd kompilacji jest nie do odczytania).
Ten sam plik kompiluje się w Unity bez błędów (`check_compile_errors` czysty).

## Przyczyna
Coplay `execute_script` kompiluje wskazany .cs **własnym Roslynem** z ograniczonym zestawem
referencji assembly. `Unity.TextMeshPro` nie jest wśród nich - każdy plik z `using TMPro;`
(albo innym typem spoza podstawowych assembly) wywala kompilację executora, mimo że projekt
jest zielony.

## Fix / obejście (zwalidowane)
Napisz mini-wrapper BEZ problematycznych usingów, który woła już skompilowaną metodę
z Assembly-CSharp-Editor (ta referencja jest dostępna):

```csharp
// scratchpad/RunShowroomSetup.cs - poza Assets/, Coplay przyjmuje absolutna sciezke
public static class RunShowroomSetup
{
    public static void Execute() { SetupFurnitureShowroom.Execute(); }
}
```

`execute_script(filePath: <sciezka wrappera>, methodName: "Execute")` - działa, bo TMPro
jest linkowane wewnątrz skompilowanego assembly projektu, a nie w kodzie kompilowanym przez executor.

## Reguła praktyczna
Skrypty edytorowe pod Coplay execute_script trzymaj wolne od `using TMPro;` / typów z paczek,
ALBO od razu odpalaj je przez wrapper. Plik wrappera może żyć w scratchpadzie (nie w Assets/) -
Unity go wtedy nie importuje.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: roslyn, execute_script, coplay
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: roslyn, execute_script, coplay
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: coplay, editor-scripting, mcp
- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] - wspolne: coplay, mcp
- [[20260611-coplay-set-property-color-json-silent-white|Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white]] - wspolne: coplay, mcp
- [[20260608-1503-mcp-scene-capture-omits-gizmos|MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
