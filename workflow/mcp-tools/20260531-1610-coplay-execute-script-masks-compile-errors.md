---
title: Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- coplay
- mcp
- execute_script
- editor-script
- compile-error
- roslyn
- workflow
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead

## Problem
Driving non-trivial C# through the Coplay MCP `execute_script` tool: when the supplied
`.cs` has ANY compile error, the call fails with a cryptic, useless message:

> Error executing script: Could not find any resources appropriate for the specified culture …
> "Microsoft.CodeAnalysis.CSharp.<…>CSharpResources.resources" was correctly embedded …

This is NOT the real error. Coplay compiles the file with its own embedded Roslyn, hits a
diagnostic, and throws while trying to *localize* the message (its satellite resource DLL is
missing). You get the same opaque exception for every compile error - a missing brace, a
CS0136 scope clash, an unknown type - so you cannot tell what's actually wrong.

## Root cause
The Coplay-embedded compiler can't format diagnostics. The exception masks the real
`CSxxxx`. Iterating blindly (guessing the bug, re-sending the whole script) burns turns.

## Solution - author the logic as a real Unity editor script, invoke via menu
1. Write the heavy logic to `Assets/Editor/MyTool.cs` as a normal `public static` class with a
   `[MenuItem("…/MyTool")]` entry method. **Unity** compiles it → real errors appear via
   `check_compile_errors` / the console (file + line + `CSxxxx`), unaffected by the Coplay bug.
2. Trigger compilation with a one-line launcher run through `execute_script` that just calls
   `AssetDatabase.Refresh()`. Then `check_compile_errors` until clean.
3. Run the tool with another trivial launcher line:
   `EditorApplication.ExecuteMenuItem("Timber Tycoon/…/MyTool")`.
4. Have the tool **write its report to a file** (e.g. `_checkpoints/report.txt`) and render any
   verification PNG itself - then Read those back. Don't rely on tool return values.

The launcher (`_FloraRun.cs` etc.) is tiny so it always compiles under Coplay; all real code
lives in Unity-compiled editor scripts. Bonus: the editor tools stay as reusable menu items.

## Gotchas that triggered the masked error here
- **CS0136**: declaring a method-scope `float x` while an earlier `foreach (var x …)` in a
  nested block also used `x`. C# forbids the name reuse across enclosing/nested scopes.
- A method-group `.Select(AssetDatabase.GUIDToAssetPath)` (overloaded) - prefer a lambda.
- Definite-assignment on a `Bounds bb;` used in an `else b.Encapsulate(...)` branch.

## Takeaway
For anything beyond a few lines, don't paste C# into `execute_script`. Put it in
`Assets/Editor/`, let Unity compile it, and invoke by menu. Keep `execute_script` for
one-liners (Refresh, ExecuteMenuItem, quick queries).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: roslyn, execute_script, coplay
- [[20260710-2252-coplay-execute-script-tmpro-compile-fail|Coplay execute_script nie kompiluje plików z `using TMPro;`]] - wspolne: roslyn, execute_script, coplay
- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] - wspolne: coplay, mcp
- [[20260611-coplay-set-property-color-json-silent-white|Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white]] - wspolne: coplay, mcp
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: coplay, mcp
- [[20260608-1503-mcp-scene-capture-omits-gizmos|MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
