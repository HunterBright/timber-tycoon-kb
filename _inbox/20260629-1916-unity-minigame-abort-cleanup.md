---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, coroutines, minigame, state-machine, camera, abort, re-entry]
date: 2026-06-29
status: draft
severity: high
---

# Aborting a coroutine-driven minigame: release the active-flag LAST, and StopAllCoroutines

## Context
A machine minigame (Unity, MonoBehaviour singleton) runs as one driver coroutine
(`SessionSequence`) that `yield return StartCoroutine(...)`s into nested helpers
(camera lerp-in, hatch/animation tweens, settle). A static `IsActive` flag both gates
player movement/camera AND is the re-entry guard for `OnInteract`. ESC routes through a
pause menu to `AbortFromExternal() → AbortMinigame()`.

Two bugs an adversarial review caught (both ship-but-corrupt-state):

## Bug 1 — releasing IsActive too early opens a re-entry window
`AbortMinigame` set `IsActive = false` at the TOP, then ran cleanup
(`AnimateHatch` + `LerpCameraBack`) asynchronously for ~1s. During that window the
re-entry guard (keyed on `IsActive`) was open → pressing the interact key launched a
SECOND session whose `EnterCamera` captured `savedCameraWorldPos = camera.position`
WHILE the old cleanup lerp was mid-flight → it saved a bogus mid-lerp pose as "home"
and restored the camera there permanently. Two camera coroutines also fought.

**Fix:** keep `IsActive == true` until cleanup truly finishes; set it `false` at the END
of the cleanup coroutine. Use a SEPARATE `bool aborting` (set at the top of
`AbortMinigame`, checked first) to prevent re-entrant aborts, instead of overloading
`IsActive` for that.

## Bug 2 — StopCoroutine(handle) does NOT stop nested coroutines
`StopCoroutine(activeSequence)` only kills the driver handle. Coroutines it started via
`yield return StartCoroutine(child)` are INDEPENDENTLY scheduled and keep running —
orphans that keep mutating the camera/hatch/drum while the cleanup coroutine drives the
same transforms. Correctness then relies fragilely on "whoever writes last wins."

**Fix:** `StopAllCoroutines()` before starting the cleanup coroutine (start cleanup
AFTER, so it survives). Transform spin driven from `Update()` (a flag, not a coroutine)
is unaffected — clear the flag explicitly.

## Rule of thumb
- One owner per transform at a time. If abort and cleanup can overlap, you have two.
- `StopCoroutine(handle)` ≠ stop the whole tree. Use `StopAllCoroutines()` (or track every
  child handle) when a coroutine fans out into nested `StartCoroutine`.
- A flag that is BOTH "system active" and "re-entry guard" will leak during async teardown.
  Split them: `IsActive` (released last) + `aborting` (guards teardown).
- Always null-guard `Camera.main`: if the lerp-in bails on a null camera but leaves
  `IsActive=true`, you get a dead session with an unlocked cursor.

## Evidence
Timber Tycoon `FertilizerMakerMinigameUI.cs` (Phase 4). Pattern cloned from
`PelletizerMinigameUI`; same class of fix applies to any of the green/red/yellow machine
minigames. Validated in Play Mode by Hunter.
