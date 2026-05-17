---
name: polybrush-iteration-rule
description: After exporting terrain mesh from generator and starting Polybrush sculpting, NEVER return to the generator. Regen overwrites the mesh asset and destroys all Polybrush work. No undo.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/asset-pipeline
  tags: [unity, polybrush, terrain, iteration, workflow]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Polybrush Iteration Rule — No Return to Generator

## What happened

1. LowPolyTerrainGenerator generates terrain mesh, exports as Unity mesh asset
2. Polybrush sculpting session: hand-carved bumps, hollows, painted vertex colors (2 hours)
3. "The noise scale is slightly off — let me tweak it in the generator"
4. Generator regenerates mesh, overwrites the mesh asset
5. All Polybrush work is gone. Polybrush has no cross-session undo.

## Rule

**Once Polybrush sculpting begins on a terrain mesh, the generator is permanently retired for that scene.** The Polybrush-sculpted mesh IS the terrain. The generator was only its origin.

```
LowPolyTerrainGenerator → export mesh asset → Polybrush sculpting START
                                                         ↑
                                          NO RETURN PAST THIS POINT
```

## Recovery

If the rule is broken: only via git (`git revert`, `git checkout` the asset file). Polybrush has no undo across sessions, only Ctrl+Z within the current Unity editor session.

## Convention

After Polybrush begins: terrain generator script parameters should not change for that scene. If experimentation is needed (testing different noise scale), do it in a separate scene on a fresh branch.

## Why it matters

Terrain is typically one of the most time-expensive manual assets in TT. The 600×650m map took ~12 hours of Polybrush work. One accidental regen = 12 hours lost.

## Workflow checkpoint

Before any generator regen: ask "does this scene have Polybrush sculpting?" If yes, create a new temporary scene for the experiment.

See also: [[polybrush-settings-low-poly]], [[backup-scene-before-modify]]
