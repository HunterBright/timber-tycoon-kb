---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, urp, lighting, day-night, reflections, reflection-probe, specular, materials]
severity: medium
time_lost: "~2 months masked by a leftover light; diagnosis itself <1h once night was truly dark"
date: 2026-06-11
status: validated
---

# Stale Skybox Reflections Whiten PBR Materials at Night (Day/Night Cycle)

## Problem
With a runtime day/night cycle, at night all URP/Lit objects (tree trunks) rendered washed-out — black albedo regions appeared light gray. Surfaces on custom diffuse-only shaders (foliage, terrain) stayed correctly dark. The artifact had been invisible for months because a leftover second directional light ("MoonLight", 0.5 intensity) kept nights bright enough to hide it.

## Root cause
Unity's **default environment reflection cubemap is generated from the skybox at bake/editor time and never updates at runtime**. A day/night cycle that animates the skybox material (or swaps ambient colors) does NOT touch the reflection cubemap — at night, every PBR material still reflects the bright daytime sky.

The reflection (GI specular) term in URP/Lit is **independent of albedo**: dielectric F0 ≈ 0.04 × fresnel × daytime-sky brightness produces a gray sheen that dominates dark albedo when scene diffuse luminance drops to night levels (~0.02–0.1). Smoothness 0.15–0.5 is enough. Custom shaders without a GI-specular term are immune — which makes the bug look like a "material/model bug on some objects" instead of a global lighting issue.

## Diagnostic signature
- Only PBR-shader objects (URP/Lit etc.) whiten; custom diffuse/unlit shaders stay dark → split follows shader family, not object type.
- Dark albedo areas lift the most (reflection is additive and albedo-independent).
- Worst at grazing view angles (fresnel).
- Check: `RenderSettings.defaultReflectionMode == Skybox`, baked `ReflectionProbe-*.exr` in the scene's lighting folder, and no `DynamicGI.UpdateEnvironment()` anywhere.

## Fixes (ranked, both verified in Play Mode)
1. **Matte materials at source**: organic/rough surfaces (bark, dirt, fabric) should have Smoothness ≈ 0 and/or Environment Reflections OFF. Fixes the worst offenders permanently, zero perf cost.
2. **Scale `RenderSettings.reflectionIntensity` with time of day** from the day/night controller (1.0 day → ~0.05 night). One line, fixes ALL PBR assets globally.
3. ~~Runtime `DynamicGI.UpdateEnvironment()`~~ — expensive stall, not worth it for stylized games.

## Meta-lesson
A "fix" light that merely brightens the scene can mask a lighting-model bug for months. When a legacy light is removed and an old artifact "returns" (user: "znowu"), suspect the artifact was always there and the light was an accidental mask — check what albedo-independent terms (specular, reflections, emission) survive in the dark.

## See also
[[urp-shadow-cascade-tuning]], [[rotating-directional-light-black-terrain]], [[unity-runtime-writes-to-shared-material-asset]]

CLAUDE.md pattern: "Legacy code conflict po refactorze" (MoonLight = legacy leftover of the pre-rewrite day/night system).
