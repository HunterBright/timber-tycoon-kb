---
type: lesson
project: Timber_Tycoon
suggested-category: engine/lessons
tags: [unity, scriptableobject, play-mode, testing, gotcha]
date: 2026-06-11
status: validated
---

# ScriptableObject changes in Play Mode DO persist after exit

## Problem
Common assumption: "Play Mode changes don't persist, everything reverts on exit." True for **scene objects** (components, transforms), **FALSE for ScriptableObject assets**. SO field edits made during Play Mode survive exiting Play Mode — the in-memory asset keeps the new value through domain reload, and a subsequent Save Project / Ctrl+S writes it to the .asset file on disk permanently.

## Why it matters
Tempting fast-test workflow: "lower the timer value in the SO during Play Mode, watch the cycle, it'll revert on exit" — it will NOT revert. Easy way to silently corrupt balance values. In a project where ALL gameplay values live in ScriptableObjects (zero magic numbers in code), every Play Mode tuning session is a potential permanent balance edit. Same family of issue as the runtime-dirtied `Mat_DayNightSkybox` material — see [[unity-runtime-writes-to-shared-material-asset]].

## Correct workflow for SO-based fast tests
1. Edit the SO value in Play Mode (works immediately, fine for the test itself).
2. After exiting Play Mode, **manually restore** the original value.
3. Do NOT Save Project before restoring.
4. Safety net: `git status` / `git diff` on the .asset file — disk only changes if Unity saved assets; restore with `git restore` if it did.

## Rule of thumb
Scene state = sandbox in Play Mode. Asset state (SO, materials, meshes) = live and shared between Edit/Play — treat every Play Mode asset edit as a real edit.

## See also
[[unity-runtime-writes-to-shared-material-asset]], [[playmode-asset-pollution-vs-disk]]
