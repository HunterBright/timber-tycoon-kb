---
title: Probe the real heightfield before scripting terrain edits — assumed profiles drift
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-26'
project: Kerf - Sawmill Tycoon
tags:
- blender
- terrain
- heightfield
- dry-run
- verification
- workflow
applies_to: []
source: ''
severity: medium
suggested-category: engine/lessons
---

# Probe the real heightfield before scripting terrain edits — assumed profiles drift

## What happened
A terrain-regrade script was specced from earlier "measured" bank heights (south bank ~5.58 dipping, north ~4.93 ≈ flush). A **dry-run that raycast the actual mesh** showed the real profile was different: the south approach was nearly flat (~5.85) with the dip living one row further in (an interpolation artifact at the un-vertexed seam), and the **north bank was 0.12 m ABOVE the deck** (needed a cut, not a fill). Applying the original spec would have raised already-correct ground, missed the real hollow, and never touched the side that was sinking the road.

## Root cause
Earlier height "measurements" were approximate / taken at points that don't coincide with the coarse vertex grid; a heightfield interpolates between sparse verts, so a value sampled at an arbitrary point ≠ the vertex heights the script actually moves. Specs built on those numbers aim at the wrong rows.

## Lesson / rule
- **Always run a compute-and-print DRY-RUN that raycasts the real mesh** (or reads the actual vertex rows) before applying ANY terrain edit. Report before→after at the exact points that matter.
- Treat the dry-run probe as ground truth and re-aim the spec against it — don't trust the brief's assumed profile.
- Watch for **asymmetry** (one bank fills, the other cuts) and for **grid-snapping** (the point you care about may have no vertex; the surface there is interpolated).
- Gate destructive terrain writes behind an `--apply` flag (dry-run by default) that backs up the source `.blend` + exported FBX before writing.

## Transferable to
Any procedural terrain/heightfield edit (Blender or Unity Terrain), road/asset conforming, abutment grading — wherever a script moves vertices based on assumed heights.
