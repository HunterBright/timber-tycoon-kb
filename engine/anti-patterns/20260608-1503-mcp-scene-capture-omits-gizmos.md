---
title: MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-06-08'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mcp
- coplay
- gizmos
- handles
- screenshot
- scene-view
- editor-tooling
applies_to: []
source: ''
promoted: '2026-07-30'
---

# MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label

## Context
Asked to deliver a "Scene-view screenshot showing gizmo labels" for 4 TreeZone objects
(each draws a wireframe box + `Handles.Label("Zone N: Lvl N (X trees)")` via a `[DrawGizmo]`
editor script). Tried to produce it programmatically.

## What was tried, and what failed
- **`mcp__coplay-mcp__capture_scene_object`** → renders only scene MESH geometry. No gizmos,
  no Handles labels. Also frames an object by its *renderer bounds*, so for a zone whose
  child trees sit far from the zone's transform pivot, it frames the trees and misses the
  gizmo entirely.
- **`mcp__unity-mcp__Unity_SceneView_CaptureMultiAngleSceneView`** → same: geometry-only
  render. Without `focusObjectIds` it frames the ENTIRE scene (here a 600×650 m map), so a
  small gizmo cluster is invisible. `focusObjectIds` needs integer instanceIDs, which the
  read tools (`get_game_object_info`, `list_game_objects_in_hierarchy`) do not return.
- **`mcp__screen-capture-mcp__take_screenshot`** (the one tool that captures the literal
  editor window, incl. gizmos/handles) → was NOT connected/registered in the session, so
  unavailable. (It's in the `unity-operator` agent's toolset, but agents share session MCP
  connectivity, so delegating wouldn't have helped.)

## Rule
Gizmos and `Handles.Label` only exist in the **Scene View GUI overlay**. MCP "capture/render"
tools that go through a camera RenderTexture (capture_scene_object, multi-angle, camera
capture) will NEVER include them - they only see geometry. To screenshot gizmos/handles you
need a **literal editor-window screen capture** (e.g. `screen-capture-mcp`), and you must
confirm that MCP server is actually connected first.

## What to do instead
1. Don't promise a gizmo screenshot unless `screen-capture-mcp` (or equivalent OS screen
   capture) is confirmed connected.
2. Verify gizmo CONTENT by data instead: read the component fields the gizmo's format string
   consumes, then state the exact label strings the gizmo will render. This is more
   authoritative than a tiny-text screenshot anyway.
3. If a visual is required, give the user a 5-second manual recipe: select the objects in
   Hierarchy → press F to frame → ensure the Scene view "Gizmos" toggle is on → screenshot.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] - wspolne: scene-view, screenshot, coplay
- [[scene-view-ab-false-positive-game-view-ground-truth|Scene View A/B screenshots gave a false-positive diagnosis - verify in the GAME view with the live camera]] - wspolne: scene-view, screenshot, mcp
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: coplay, mcp
- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: coplay, mcp
- [[20260710-2252-coplay-execute-script-tmpro-compile-fail|Coplay execute_script nie kompiluje plików z `using TMPro;`]] - wspolne: coplay, mcp
- [[20260710-2250-unity-autonomous-smoke-runner-flag-file|Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
