---
name: scene-attachment-check-before-deleting-monobehaviour
description: Before deleting a MonoBehaviour script, grep all scenes + prefabs + assets for references. Deleting in-use scripts leaves "missing script" errors requiring manual cleanup.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, unity, scene-safety, deletion]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects, claude-code-projects]
---

# Scene Attachment Check Before Deleting MonoBehaviour

## What happened

Deleting a MonoBehaviour script that was still referenced in scene objects leaves Unity with "missing script" components. Each missing reference requires manual removal via Inspector → Remove Component. With 10+ attached objects, this is a half-day cleanup.

## Rule

Before deleting any MonoBehaviour script file, run the full attachment check:

```bash
# 1. Find code references (other scripts inheriting or referencing this class)
grep -r "ClassName" Assets/Project/Scripts/

# 2. Find scene references (most critical)
grep -r "ClassName" Assets --include="*.unity"

# 3. Find prefab references
grep -r "ClassName" Assets --include="*.prefab"

# 4. Find ScriptableObject references (stored as class name in YAML)
grep -r "ClassName" Assets --include="*.asset"
```

Only if all 4 return empty: proceed with deletion.

## Why this matters

"Delete refactored code" is one click. "Fix 47 missing scripts in scene" is half a day. Asymmetric cost.

Unity scenes store MonoBehaviour references by script GUID. When the script is deleted, GUID becomes dangling. Unity doesn't error immediately — it silently shows "Missing Script (MonoBehaviour)" in Inspector, but the scene still loads. Problems surface later when something tries to call the missing component.

## When recovery is needed

If you already deleted and have missing scripts: see [[scene-files-binary-never-edit]] for binary scene editing risk. Safest fix: open scene, select each affected object, Inspector → find missing component → Remove Component. With many objects, write an Editor script to automate cleanup.

## Applies to

Claude Code agents performing refactors that move or merge classes. Add to pre-delete checklist in any migration sprint (see [[before-delete-legacy-class-checklist]]).

See also: [[before-delete-legacy-class-checklist]], [[scene-files-binary-never-edit]], [[backup-scene-before-modify]]
