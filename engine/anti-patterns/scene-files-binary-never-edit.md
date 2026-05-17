---
name: scene-files-binary-never-text-edit
description: Unity .unity and .prefab files look like YAML but contain internal GUIDs — text editing breaks scene. Recovery via Inspector Remove Component.
metadata:
  type: anti-pattern
  project: timber-tycoon
  suggested-category: engine/anti-patterns
  tags: [unity, scene, yaml, prefab, missing-script]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ANTI-PATTERN: Scene Files Are Binary — Never Edit as Text

## The Trap
Unity `.unity` scene files and `.prefab` prefab files open in any text editor as YAML-like text. They look editable. You try to "quickly fix" a component reference, rename a class, or move a GameObject by editing the text. The scene then fails to load or has missing/corrupted GameObjects.

## Why It Fails
Unity scene/prefab files contain auto-generated GUIDs and `fileID` references that link Unity objects internally. A single wrong line can:
- Break a GUID reference → component becomes "Missing Script"
- Corrupt file format → scene refuses to load
- Silently lose GameObjects → they disappear from hierarchy with no error

These errors often aren't noticed until reopening the scene, by which time the damage is in version control.

## What to Do Instead

**Renaming a component class:** Use IDE rename (ReSharper/Rider) or update the class name in code; Unity auto-updates `.meta` GUID references. Never rename by text-editing the scene file.

**Moving GameObjects between scenes:** Drag in Unity Hierarchy. Never cut-paste YAML blocks.

**Fixing missing scripts:** Script file was deleted but scene still references it:
1. Open scene → look for red "Missing Script" warnings in Console
2. Select the affected GameObject → Inspector → find "Missing Script" slot
3. Click ⋮ menu → Remove Component
4. Re-add the correct component

**Bulk read operations (acceptable):** `grep "MyClass" *.unity` to find all scene files referencing a class — read-only, no edits. Also fine for source-control diff review.

**Force text serialization for diffability:** Edit → Project Settings → Editor → Asset Serialization → Force Text. This makes diffs readable in git, but still: don't edit the files by hand.

## Recovery
If a scene is already broken by text edit and won't load:
1. `git diff HEAD -- Assets/Demo_Scene.unity` → see what changed
2. `git checkout HEAD -- Assets/Demo_Scene.unity` → restore last working version
3. Redo the intended change via Unity Inspector

This is why the project rule is: **backup scene before any modification** (see [[never-destructive-ops-in-play-mode]]).

## See also
[[runtime-vs-editor-script-separation]], [[never-destructive-ops-in-play-mode]]
