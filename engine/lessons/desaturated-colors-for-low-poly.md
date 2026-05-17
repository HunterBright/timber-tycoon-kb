---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [color, low-poly, aesthetic, terrain, immersion]
severity: medium
time_lost: ""
date: 2026-05-17
status: draft
applies_to: ["unity-projects", "blender-pipelines"]
---

# Desaturated Colors for Low-Poly Aesthetic

## Problem
First instinct for low-poly is saturated, bright colors (think mobile game assets). The result looks cheap — players say "it feels unpolished" without being able to articulate why. The palette is the problem.

## Root cause
Saturated low-poly = "mobile shovelware" visual language. Desaturated, slightly muted palette = "premium indie" (Slime Rancher, Townscaper, A Short Hike). The difference is entirely in the palette choices.

## Solution
Use desaturated, slightly cool-toned palettes:
- Grass: avoid `#5B9947` (too bright); prefer `#3D6B35` to `#4A7841`
- Rock: avoid `#8C806B` (too warm); prefer `#5A524A` (cooler, muted)
- Sand/dirt: avoid `#D1C49E` (jaundiced); prefer `#8B7D6B` (natural beige)
- Color variation per vertex: keep LOW (~0.03 jitter), not high (0.06+ = patchy/"pstrokaty" effect)
- Always test in-context: Play Mode at player eye height, NOT Scene View flyover

## What didn't work
High-saturation palette — looks cheap at player eye level even if it looks fine in the aerial editor view.

## Transferability
Applies to any stylized low-poly game targeting "cozy/premium indie" aesthetic. The desaturation principle holds for isometric builders, walking sims, farming games, and open-world explorers.

## Related
- [Vertex color gamma correction](vertex-color-gamma-correction-blender-to-unity.md)
