---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, play-mode, git, scene, materials, day-night-cycle, save-hygiene]
date: 2026-06-08
status: validated
---

# Play-Mode in-memory edits pollute on-disk assets — and a "fix" can produce zero git diff

## Severity
Medium — causes confusing git state and risks committing/saving the wrong thing during safety commits.

## Symptom
During a safety-commit exercise the open scene appeared to have a machine in the wrong
gated state (`MachineController.activeOnStart = True` on Pelletizer), and two materials
(`Mat_DayNightSkybox.mat`, `Mat_Lamp_Bulb.mat`) kept showing as modified in `git status`
even after a clean `git checkout --`.

## Root cause
A prior Play-Mode session (with debug cheats activating machines) mutated **in-memory**
object/material state. A runtime system — here the day/night cycle — writes to `.mat`
assets at runtime; those writes persist to disk and dirty git. Meanwhile the scene's
on-disk value was already correct; only the in-memory copy was polluted.

## Two non-obvious consequences
1. **A corrective edit that restores the on-disk value yields NO git diff.** I set
   Pelletizer `activeOnStart = false` and saved the scene — but `Demo_Scene.unity` showed
   zero change vs HEAD, because the committed scene already held `false`. The `True` only
   ever existed in memory. Don't force a phantom file into a commit to satisfy a plan; if
   there's no diff, the asset was already correct.
2. **Saving the scene can re-dirty runtime-mutated materials** (the save flush touches
   them again), so material pollution can reappear right after you save — even in Edit Mode.

## What to do
- Before a safety commit, **verify uncommitted state with git, don't trust assumptions** —
  `git add -A` checkpoint commits may have already captured the "uncommitted" work, and a
  supposed dirty tree can be almost clean.
- Treat runtime-written assets (skybox/lamp materials under a day/night cycle) as
  **expected pollution**; exclude them from selective commits rather than fighting them.
  (Better: eliminate the writes at source — see [[unity-runtime-writes-to-shared-material-asset]].)
- When a plan says "commit file X" but `git diff` shows no change for X, that's a signal the
  asset was already correct — report it, don't fabricate a change.

## Generalizes to
Any Unity project where a runtime system mutates serialized assets (materials, scriptable
objects) during Play Mode, plus any "snapshot working tree" checkpoint-commit workflow.

## See also
[[unity-runtime-writes-to-shared-material-asset]], [[scriptableobject-playmode-persistence]]
