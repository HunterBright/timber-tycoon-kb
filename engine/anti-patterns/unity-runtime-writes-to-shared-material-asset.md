---
title: 'Anti-pattern: runtime writes to a shared material ASSET'
type: anti-pattern
status: verified
confidence: high
verified: ''
date: '2026-06-11'
project: Kerf - Sawmill Tycoon
tags:
- unity
- materials
- serialization
- git
- day-night-cycle
- materialpropertyblock
applies_to: []
source: ''
suggested-category: engine/anti-patterns
---

# Anti-pattern: runtime writes to a shared material ASSET

## Problem

A day/night cycle script wrote `_TimeOfDay` / `_SunDirection` via `material.SetFloat/SetVector` on the **shared material asset** (`Mat_DayNightSkybox.mat`). In the editor, writing to `renderer.sharedMaterial` (or a material referenced directly from an asset, e.g. `RenderSettings.skybox`) mutates the serialized asset itself.

## Symptoms

- The `.mat` file shows as modified in git after **every** editor play session.
- The "change" is just the last-written runtime values (time of day, sun direction) - pure noise.
- Easy to accidentally commit; with LFS the diff is opaque (pointer-only), so the noise looks like a real change.

## Why it happens

Unity persists edits made to material assets in Edit Mode AND in Play Mode when the object written to is the asset, not an instance. Play Mode does not sandbox asset objects - only scene objects are restored on exit.

## Correct approaches (both Play-Mode-verified in Timber Tycoon)

1. **Runtime material instance** - `new Material(sharedMat)` at startup, assign it, write only to the copy. **Required for `RenderSettings.skybox`**: the skybox has no Renderer, so MaterialPropertyBlock does NOT apply there. This is the verified fix for `Mat_DayNightSkybox`.
2. **MaterialPropertyBlock** - per-renderer overrides, never touches the asset. The verified fix for every Renderer-based case (e.g. per-instance emission pulses, body-color randomization).
3. **Global shader properties** - `Shader.SetGlobalFloat/Vector` when the value is scene-wide (sun direction is a perfect fit); nothing serialized at all.

## Teardown hygiene

Whatever you create or override at runtime, clean it up on teardown: `Destroy()` the runtime material instance (it leaks otherwise), clear MaterialPropertyBlocks if the renderer outlives the effect, and reset global shader properties that would leave the editor in a misleading state. The goal is that exiting Play Mode leaves zero trace - in memory and in `git status`.

## Detection rule

If a `.mat` keeps reappearing as dirty in git after play sessions, diff the actual YAML (with LFS: `git show HEAD:file | git lfs smudge | diff - file`) - if only animated-looking floats/vectors changed, some script is writing to the shared asset.

## See also

[[scriptableobject-playmode-persistence]], [[playmode-asset-pollution-vs-disk]], [[four-phase-weighted-smoothstep-day-night]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[playmode-asset-pollution-vs-disk|Play-Mode in-memory edits pollute on-disk assets - and a "fix" can produce zero git diff]] - wspolne: day-night-cycle, git, materials
- [[20260713-2020-unity-binary-scene-in-git-lfs-is-irreversible-bloat|Anti-pattern: binarna scena Unity w Git LFS = nieodwracalny, rosnacy bez konca bloat]] - wspolne: git, serialization
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] - wspolne: serialization, materials
<!-- /POWIAZANE:auto -->
