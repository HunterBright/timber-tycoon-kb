---
name: tripo-asymmetric-floating-retopo
description: Tripo generates asymmetric models (left/right different) and floating elements. Fixes in Blender: Mirror modifier for symmetry, Snap + Merge by Distance for floating pieces.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/asset-pipeline
  tags: [tripo, blender, retopo, asymmetry, cleanup]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [blender-pipelines]
---

# Tripo Asymmetric / Floating Elements — Blender Retopo

## Known Tripo weaknesses

1. **Asymmetry:** left and right sides of symmetric objects generated differently (different wheel shapes, uneven door panels, mismatched legs)
2. **Floating elements:** loose details (bolts, handles, panels) not attached to main mesh — visible as disconnected pieces when rotating in viewport

## Fix: Asymmetry via Mirror modifier

```
Step 1: Identify which half is "good" (usually right side for vehicles)
Step 2: Edit Mode → select and delete the "bad" half
         (loop select along center axis, delete faces)
Step 3: Object Mode → Add Modifier: Mirror (X-axis by default)
Step 4: Apply modifier (converts mirror to geometry)
Step 5: Edit Mode → Select All → Mesh → Merge by Distance (threshold: 0.001m)
         This welds duplicate vertices at centerline
```

**Threshold note:** 0.001m (1mm) merge distance catches rounding drift from Tripo's generation without accidentally merging intended gaps.

## Fix: Floating elements via Snap + Merge

```
Step 1: Enter Edit Mode → rotate view to identify floating piece
         (disconnected geometry orbits separately — visible immediately)
Step 2: Select floating piece vertices
Step 3: G → move toward nearest main mesh face
Step 4: Enable Snap: Vertex → Nearest (magnet icon, shortcut: Shift+Tab)
Step 5: Snap floating vertices to main mesh vertices
Step 6: Mesh → Merge by Distance (0.001m) to weld
```

## When to skip

Low-priority background props that player won't examine closely:
- Asymmetry fix: skip if symmetry difference < 5% and player views from distance
- Floating fix: can't skip (floating pieces fall off / show separately in Unity if not merged)

## Time investment

- Asymmetry fix: 10-20 minutes per model
- Floating fix: 5-15 minutes per element

At TT's quality bar: always fix floating, fix asymmetry on interactive/inspectable assets, skip asymmetry on distant props.

## General lesson

Tripo's strengths: speed (3 minutes to concept), organic shapes, creative variation. Tripo's known weaknesses: symmetry, physics-accurate attachments, precise dimensions. Manual Blender fixes take 20 minutes and turn "AI slop" into production asset.

See also: [[tripo-cleanup-pipeline]], [[tripo-vocab-firewood-wedge]], [[zero-floating-zero-flickering-mandate]]
