---
title: MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-06'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mcp
- coplay
- scene-capture
- prefab-stage
- screenshot
- qa
- scene-view
applies_to: []
source: ''
severity: low
promoted: '2026-07-30'
---

# MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage

## Problem
To visually QA a single model in isolation (clean neutral backdrop, no scene clutter), the intuitive move is: open the prefab in Prefab Mode, then call a capture tool. With the Coplay/Unity MCP capture tools this DOES NOT work - the returned screenshot shows the active main scene at the requested camera pose, ignoring the open prefab stage entirely.

## What happened
- `PrefabStageUtility.OpenPrefab(path)` succeeded (returned a valid stage, correct bounds logged).
- Camera was aimed at the prefab-stage content center (~origin).
- `mcp__coplay-mcp__capture_scene_object` (no `gameObjectPath`) returned a shot of the **main scene** at world-origin (terrain + a fence post) - NOT the prefab. The prefab content was nowhere in the image.

## Root cause
These capture tools render the active SCENE through a temporary camera. A Prefab stage is a separate editing context with its own preview scene; the capture path does not target it. So the camera pose is honored but applied to the main scene, not the stage.

## Working recipe (verify a model in isolation-ish)
Drive the SceneView camera around the model's **scene instance** instead:
1. `GameObject.Find("<RootName>")` in the active scene; encapsulate all child `Renderer.bounds` into one `Bounds b`.
2. `var sv = SceneView.lastActiveSceneView;` (fallback to `SceneView.sceneViews[0]`).
3. Per angle: `sv.LookAt(b.center, Quaternion.Euler(pitch, yaw, 0), b.extents.magnitude * ~1.0f); sv.Repaint();` then call `capture_scene_object` (no path).
4. Cover 6 sides with yaw 0/90/180/270 (+ high pitch for top-down). `Quaternion.Euler(pitch,180,0)` puts the camera on the +Z side looking −Z, etc.

This gives controlled, framed, single-object shots that come back to the caller's own context (so you can eyeball them directly, not via an agent's text summary).

## Trade-off
The model appears against the real scene background (e.g. sawmill wall, mountains), not a neutral backdrop. That's acceptable for a missing-face / flipped-normal check: a hole in a surface that should be solid still reads as an obvious shading discontinuity against any background. If a truly clean backdrop is required, temporarily instantiate the prefab at an empty spot in the scene (EDIT mode, never Play mode), capture, then delete - but that modifies the scene (back it up first).

## What didn't work
- Prefab Mode + capture → returns main scene (this lesson).
- `capture_scene_object` does have a `gameObjectPath` arg that frames a named scene object, but it only yields one direction; for multi-angle coverage you still drive the SceneView camera yourself.

## Transferability
Any Unity project using Coplay/Unity MCP for visual QA of a model/prefab: asset re-import validation, normal/face checks, material spot-checks, before/after screenshots.

## Related
- [[fbx-export-standard-settings-blender-to-unity]]
- [[20260606-1632-in-place-fbx-overwrite-static-vs-rigged]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260608-1503-mcp-scene-capture-omits-gizmos|MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label]] - wspolne: scene-view, screenshot, coplay
- [[scene-view-ab-false-positive-game-view-ground-truth|Scene View A/B screenshots gave a false-positive diagnosis - verify in the GAME view with the live camera]] - wspolne: scene-view, screenshot, mcp
- [[20260611-coplay-set-property-color-json-silent-white|Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white]] - wspolne: coplay, mcp
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: coplay, mcp
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] - wspolne: coplay, mcp
- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] - wspolne: coplay, mcp
<!-- /POWIAZANE:auto -->
