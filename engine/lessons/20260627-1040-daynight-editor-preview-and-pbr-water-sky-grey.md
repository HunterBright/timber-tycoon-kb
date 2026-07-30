---
title: Stylized PBR water looks great in editor but grey in-game - the day/night cycle drives lighting only at runtime
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-27'
project: Kerf - Sawmill Tycoon
tags:
- urp
- water-shader
- day-night-cycle
- ambient
- reflection-probe
- editor-vs-runtime
- pbr
- stylized-water
applies_to:
- unity
- urp
- water
- lighting
source: ''
severity: medium
time_lost: ~1h (two misleading editor previews before diagnosing)
promoted: '2026-07-30'
---

# Stylized PBR water looks great in editor but grey in-game - the day/night cycle drives lighting only at runtime

## Problem
A custom transparent URP water shader (river+lake) looked a clean translucent blue in SceneView editor captures, but in actual Play Mode at morning (07:34) the water was a dull **grey** sheet from the player's low eye-level angle. Separately, a waterfall shader switched to the same PBR lighting went **near-black in shade** even though it looked fine in editor. Two iterations were approved off editor previews, then rejected the moment they were seen in-game.

## Root cause
Two compounding issues:

1. **Editor preview ≠ runtime lighting.** A `DayNightCycle` MonoBehaviour wrote `RenderSettings.ambientSkyColor/Equator/Ground` (Trilight), `RenderSettings.reflectionIntensity`, and the directional light's rotation/color/intensity **only in `Update()` / `Start()` - i.e. only in Play Mode**. In edit mode those RenderSettings are whatever is serialized in the scene, so SceneView shows a *different* sky/ambient/sun than the running game. Every material preview in the editor was lit by the wrong state.

2. **PBR water mirrors the pale daylight sky.** `UniversalFragmentPBR` adds a glossy environment reflection scaled by `smoothness` and `RenderSettings.reflectionIntensity`. The reflection cubemap is a *frozen daytime sky* (~grey-blue). At a low/grazing viewing angle (player standing at water level), the fresnel-weighted environment reflection dominates the surface → the blue base is washed to grey. The more "life" added via smoothness, the greyer it gets. Top-down editor captures hide this because fresnel is low there.

   The waterfall going black was the mirror image: its PBR `InputData.bakedGI` was left at 0, so it received **no ambient diffuse** and collapsed to black wherever the sun was occluded.

## Solution
- **Preview in the runtime lighting state.** Wrote a throwaway editor tool that replicates the cycle's computed values for a target time-of-day (set the same `ambientSky/Equator/Ground`, `reflectionIntensity`, sun rotation/color/intensity), captures, then restores. Iterate against *that*, never the default editor lighting.
- **Stop the sky-mirror greyness:** drop water `_Smoothness` low (0.42 → ~0.14). Get "texture/life" from animated **albedo** flow streaks (procedural noise in-shader), not from gloss/reflection.
- **Never let it go black + keep time-of-day consistency:** set `inputData.bakedGI = SampleSH(normalWS);` before `UniversalFragmentPBR`. SH picks up the cycle's Trilight ambient, so both surfaces tint with the time-of-day sky and stay lit in shade. Apply the *same* line to any sibling shader (waterfall) you want to react identically.

## What didn't work
- Deepening base color + cutting smoothness to 0.25 earlier: killed the grey but also killed all life (flat). The fix is low smoothness **plus** albedo-based animated texture, not just low smoothness.
- Judging any of it from editor SceneView at the default (non-runtime) lighting - produced two false "looks great" calls.

## Transferability
Any Unity/URP project with (a) a script-driven day/night cycle that writes `RenderSettings`/light at runtime, and (b) stylized transparent water or other reflective surfaces. The two rules generalize: **always evaluate runtime-lit materials in the runtime lighting state, not the editor default**, and **stylized water should hold its own albedo color (low smoothness + SH ambient), not mirror the sky.**

## Related
- [[four-phase-weighted-smoothstep-day-night]]
- [[low-poly-water-side-wave]]
- [[vertex-color-gamma-correction-blender-to-unity]]
- [[never-destructive-ops-in-play-mode]]
