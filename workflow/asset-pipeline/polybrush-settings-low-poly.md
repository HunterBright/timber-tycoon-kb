---
title: Polybrush Settings for Low-Poly Terrain
type: lesson
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- polybrush
- terrain
- settings
- low-poly
applies_to:
- unity-projects
source: ''
description: Polybrush settings for TT low-poly terrain - Outer Radius 40, Sculpt Power 30, Falloff linear, Strength 1.0. Default settings produce sharp bumps incompatible with low-poly style.
severity: medium
suggested-category: workflow/asset-pipeline
name: polybrush-settings-low-poly
---

# Polybrush Settings for Low-Poly Terrain

## Why settings matter

Default Polybrush settings produce sharp, spiky bumps - wrong for TT's low-poly aesthetic. TT terrain needs smooth, low-frequency undulation (gentle hills, shallow valleys). Settings must match the aesthetic before sculpting begins.

## Tuned settings for TT

**Sculpt tool:**
| Setting | Value | Reason |
|---------|-------|--------|
| Outer Radius | 40 | Large brush = low-frequency variation, smooth hills |
| Inner Radius | 20 | 50% of outer = natural falloff gradient |
| Sculpt Power | 30 | Moderate height change per click (not too aggressive) |
| Falloff Curve | Linear | No abrupt edges, fits low-poly soft style |
| Strength | 1.0 | Full effect per stroke |

**Smooth tool (for softening harsh transitions):**
| Setting | Value |
|---------|-------|
| Outer Radius | 30 |
| Strength | 0.5 |

**Vertex color paint tool:**
| Setting | Value |
|---------|-------|
| Outer Radius | 20 |
| Falloff | Soft (not linear) - for blended transitions between grass/dirt/rock |
| Strength | 0.4 |

## Post-Polybrush

Use Mesh → Smooth Vertex (custom Editor tool in TT) for additional softening of harsh transitions. Runs a Laplacian smooth pass over the mesh. Run 2-3 times on areas that still feel sharp.

## What "wrong" settings look like

Default Polybrush: Outer Radius 5, Power 80 → produces small sharp protrusions that look like inverted spikes. The low-poly style amplifies this - each vertex moves dramatically with small radius.

With tuned settings (Radius 40, Power 30): vertices move subtly across a wide area → smooth rolling hills that match TT's established terrain aesthetic.

## Save as preset

Polybrush supports brush presets (toolbar dropdown). Save these settings as "TT Low-Poly Sculpt" and "TT Low-Poly Smooth" to avoid re-entering per session.

See also: [[polybrush-iteration-rule]], [[backup-scene-before-modify]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[polybrush-iteration-rule|Polybrush Iteration Rule - No Return to Generator]] - wspolne: polybrush, terrain
- [[20260628-1105-lowpoly-lake-shore-jagged-fix|Low-poly lake shore looks jagged (serrated) - submerge the rim + widen the water, don't densify]] - wspolne: terrain, low-poly
- [[desaturated-colors-for-low-poly|Desaturated Colors for Low-Poly Aesthetic]] - wspolne: terrain, low-poly
- [[terrain-skirt-against-seethrough-gap|Terrain Skirt Against the See-Through Gap]] - wspolne: terrain, low-poly
- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] - wspolne: terrain, low-poly
<!-- /POWIAZANE:auto -->
