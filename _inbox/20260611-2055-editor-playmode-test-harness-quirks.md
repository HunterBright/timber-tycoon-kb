---
type: lesson
project: Timber_Tycoon
severity: medium
suggested-category: engine/lessons
tags: [unity, editor-scripting, playmode, automation, testing, domain-reload]
date: 2026-06-11
status: draft
---

# Editor-driven Play Mode test automation — three engine quirks that break naive harnesses

Context: an editor state machine driving N sequential real Play Mode sessions
(enter play → act via runtime APIs → save → exit play → repeat), state in `SessionState`
(survives domain reloads). Three failures took a full diagnostic cycle to pin down:

## 1. Unity fires DUPLICATE `playModeStateChanged` transitions
`EnteredEditMode` can fire more than once per actual transition. A handler that does
"advance to next session" on each event silently **skips sessions** (we saw 1 → 3).
Fix: consume the per-session completion flag atomically (read + immediately reset) so a
duplicate event sees "not completed" and lands in a harmless retry path instead of advancing.

## 2. `EditorApplication.update` ticks are NOT game frames
Editor update fires ~10× more often than the player loop renders frames (measured:
frameCount=6 after 120+ editor ticks). Counting ticks to wait for game-time animations
(e.g. a 0.5 s shrink) fails: the wait elapses before the animation ran.
Fix: condition-based waits with wall-clock timeout —
`while (!cond && timeSinceStartup - t0 < timeout) yield return null;` — never tick counts.

## 3. `EditorApplication.delayCall` is LOST on domain reload
Scheduling `EnterPlaymode` via delayCall between sessions stalls the whole run forever if a
recompile/domain reload happens in between (e.g. an MCP tool writing a script file mid-run).
Fix: (a) self-heal in the `[InitializeOnLoad]` static ctor — if a run is active and the editor
is idle in edit mode, reschedule; (b) an edit-mode watchdog in the update loop that re-enters
play after ~15-30 s of unexpected idling.

## General shape that ended up robust
- All cross-session state in `SessionState` (survives reloads, clears on editor restart).
- Per-session "completed" flag set only when the step iterator finishes; uncommitted sessions
  RETRY instead of being skipped (cap retries).
- Results appended to a file on disk after every assertion — survives any crash and lets an
  outside process poll progress (status file with timestamps = liveness signal).
- Back up the player's save slot before the run, restore in `Finish()` and in an abort menu item.
- Each play session is ~5 min wall-clock in a mid-size project (domain reload dominates) —
  budget accordingly.
