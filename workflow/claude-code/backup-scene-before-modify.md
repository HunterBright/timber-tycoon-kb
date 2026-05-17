---
name: backup-scene-before-modify
description: Before any structural scene modification, copy scene file to Assets/_Backup_{YYYY-MM-DD}/. Date-stamped per session. Three safety nets: git + scene backup + Unity undo.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, unity, backup, scene-safety]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects, claude-code-projects]
---

# Backup Scene Before Structural Modification

## Rule

Before any structural modification to a scene file: copy it to a date-stamped backup folder.

```bash
# PowerShell (Windows)
$date = Get-Date -Format "yyyy-MM-dd"
$backupDir = "Assets/_Backup_$date"
New-Item -ItemType Directory -Force $backupDir | Out-Null
Copy-Item "Assets/Demo_Scene.unity" "$backupDir/Demo_Scene.unity"
```

## When backup is required

- Bulk add/remove components across many GameObjects
- Deleting GameObjects from hierarchy
- Restructuring hierarchy (re-parenting, ordering changes)
- Any Coplay MCP batch operation on scene objects
- Running Editor scripts that modify scene via `Undo.RecordObject` + `EditorUtility.SetDirty`

## When backup is NOT required

- Minor Inspector tweaks to single objects (Unity's own Ctrl+Z handles this)
- Material swaps
- Position adjustments on individual objects

## Defense in depth

Three safety nets work together:
1. **Unity undo (Ctrl+Z):** Works within editor session. Lost on editor restart.
2. **Scene backup:** Survives editor restart. Recoverable manually (copy file back).
3. **Git:** Survives machine restart. Recoverable via `git checkout`. Requires remembering to commit.

Cost of backup: 1 second. Cost of scene disaster without backup: hours.

## Retention

Keep last 7 days of backups. Prune older:
```bash
# PowerShell
Get-ChildItem "Assets" -Filter "_Backup_*" -Directory | 
    Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-7) } |
    Remove-Item -Recurse -Force
```

See also: [[scene-attachment-check-before-deleting-monobehaviour]], [[scene-files-binary-never-edit]]
