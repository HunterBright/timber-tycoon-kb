---
name: debugging-search-first-trust-render-check-upstream
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [debugging, methodology, web-search, ground-truth, root-cause, process]
severity: high
date: 2026-05-31
status: validated
---

# Debugging methodology: search-first, trust the render, check for upstream sabotage

## Context
A single defect — an NPC mesh that rendered mangled/T-posed — ate most of a session because the investigation broke three process rules. Each rule below is stated tool-agnostically (it applies to any engine/tool/project) and is justified by exactly what went wrong this time. The concrete case lives in [[fbx-binary-overwrite-corrupts-bindposes]] and [[discriminating-clip-vs-rig-vs-skin-humanoid-defect]].

## Rule 1 — Search-first after ~2 failed attempts
When a hypothesis fails, or a fix doesn't work after **~2 attempts**, **web-search the exact symptom before generating another memory-based hypothesis.** Mature tools (Unity, Blender, Mixamo, URP, …) have well-trodden traps that are almost always documented. This session we re-derived a documented trap — "binary-overwriting an FBX corrupts the rig/bind" — empirically, across many turns, instead of searching for it. The cost of one search is far below the cost of another speculative fix-and-test loop. (Project corollary: also [read the actual code/asset before hypothesizing](../../workflow/claude-code/read-actual-code-before-hypothesizing.md).)

## Rule 2 — When a metric disagrees with the render, the render is ground truth
If a measurement, report, or log **disagrees with what is actually rendered/observed, the rendered result wins and the measurement is suspect.** We chased bone-space numbers (a "foot asymmetry") for several turns while the mesh on screen was telling the real story: the bones were fine, the *skin* was broken. A metric can be over-sensitive, can measure the wrong layer, or can be computed on a representation that the user never sees. Never keep tuning the thing the numbers point at if your eyes say something else is wrong — go find the layer the render is complaining about.

Refinement: WHICH render is ground truth also matters — Scene-View captures can themselves mislead for runtime effects like shadows; see [[scene-view-ab-false-positive-game-view-ground-truth]].

## Rule 3 — Before escalating, check that an earlier step isn't sabotaging this one
Before declaring a defect unfixable, escalating, or rebuilding from scratch, **verify that a prior step isn't silently breaking the current approach** — an earlier file overwrite, an import setting, a cached artifact, a stale config. The hidden saboteur this session was a binary FBX overwrite done *much earlier* "to keep a GUID"; every later fix was applied on top of already-corrupted data, so nothing could work. When every reasonable local fix fails, widen the time window and audit the inputs, not just the current operation.

## Transferability
These are project-agnostic process rules, not Unity facts. Any debugging effort — game engine, backend, data pipeline — benefits: cap speculation with a search, anchor truth to the observable result rather than the instrument, and audit upstream inputs before concluding the current step is at fault. The common failure mode they prevent is a long, confident chase down the wrong layer.

## Related
- [[fbx-binary-overwrite-corrupts-bindposes]] (the upstream saboteur — still in _inbox, pending promotion)
- [[discriminating-clip-vs-rig-vs-skin-humanoid-defect]] (where the metric misled us — still in _inbox, pending promotion)
- [Read actual code before hypothesizing](../../workflow/claude-code/read-actual-code-before-hypothesizing.md)
- [[scene-view-ab-false-positive-game-view-ground-truth]]
