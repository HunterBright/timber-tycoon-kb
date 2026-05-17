---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, play-mode, scene-safety, data-loss, claude-code]
severity: high
time_lost: "hours (scene rebuild)"
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# NEVER save_scene or DestroyImmediate in Play Mode

## Problem
Calling `save_scene` (via Coplay MCP or editor scripts) while Unity is in Play Mode overwrites the actual edit-mode scene with Play Mode garbage state. `DestroyImmediate` in Play Mode destroys prefab references permanently. Both cause unrecoverable data loss.

## Root cause
- Unity Play Mode is sandboxed: runtime state does NOT persist unless explicitly saved
- `save_scene` in Play Mode saves the CURRENT runtime scene — including runtime-generated objects, destroyed objects, modified transforms — as if it were the designer's intended scene
- `DestroyImmediate` in Play Mode destroys both the runtime instance AND the underlying prefab reference in the asset database

## Solution
- Always check `EditorApplication.isPlaying` before calling `save_scene` or `DestroyImmediate`
- Backup scene before any structural modification: copy `Assets/Demo_Scene.unity` to `Assets/_Backup_{date}/Demo_Scene.unity`
- Recovery: via git. If no commit, scene is gone.
- Coplay MCP agents must check Unity state before all destructive operations

## What didn't work
Trusting that Play Mode operations "just don't save." They do, if you explicitly call save.

## Transferability
Hard rule for every Unity project. Any CI/CD automation or AI agent that modifies Unity scenes must implement this check. The rule is the same regardless of project, engine version, or team size.

## Related
- [Backup scene before modify](../../workflow/claude-code/backup-scene-before-modify.md)
- [Scene files are binary, never text-edit](../anti-patterns/scene-files-binary-never-edit.md)
