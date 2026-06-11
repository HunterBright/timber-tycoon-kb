---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, debugging, scene-view, game-view, shadows, screenshots, mcp, methodology]
severity: high
date: 2026-06-11
status: validated
---

# Scene View A/B screenshots gave a false-positive diagnosis — verify in the GAME view with the live camera

## What happened
Diagnosing a moving shadow band on roads, a Scene-View A/B capture test (toggle a suspect, screenshot, compare) **confidently pointed at distant mountain shadow casters** — at a Scene-View sun elevation of ~4° the mountains visibly produced the artifact. The conclusion was wrong: the real cause was terrain self-shadow leaking onto near-coplanar road meshes, and it only revealed itself via **in-game runtime toggles at sun elevation ~40°** in the Game view with the live player camera (see [[urp-distant-caster-shadow-band]] for the full diagnosis). The mountain fix was valid hygiene but secondary — it only ever mattered at sun <8°.

## Why Scene View lied
Shadow rendering is **camera-relative**: cascade splits, shadow distance, and texel precision all depend on the rendering camera's position and view. The Scene-View camera is NOT the player camera — it sits elsewhere, frames differently, and produces a different cascade/precision layout. An artifact whose boundary is camera-relative (like a distance-dependent precision leak) can appear, disappear, or appear with a *different apparent cause* in Scene View. On top of that, the A/B was run at a different sun elevation than the one the player reported.

## The refinement of "the render is ground truth"
[[debugging-search-first-trust-render-check-upstream]] establishes: when a metric disagrees with the render, trust the render. This lesson adds: **WHICH render matters.** The ground-truth render is the one the player sees — the Game view, through the live gameplay camera, under the reported runtime conditions (time of day, position). Any other render — Scene View, editor captures, offscreen RenderTextures — is an instrument, and instruments can mislead exactly like metrics.

## Additional dead end worth remembering
**Edit-Mode offscreen RenderTexture renders show NO real-time shadows at all.** A scripted Camera.Render() to a RenderTexture in Edit Mode produced shadow-free images — useless for any shadow investigation, and dangerously close to "the artifact is gone" if you don't notice ALL shadows are missing.

## Rule
For runtime-dependent rendering effects (shadows, LOD, fog, post-processing, anything camera- or time-dependent): A/B test **in Play Mode, in the Game view, with the live camera, under the reported conditions**. Scene-View screenshots are for geometry and layout questions, not for runtime lighting verdicts. The cheapest reliable instrument is a runtime toggle harness the player drives at the bug spot — described in [[urp-distant-caster-shadow-band]].

## See also
[[urp-distant-caster-shadow-band]], [[debugging-search-first-trust-render-check-upstream]]
