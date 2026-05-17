---
name: polybrush-settings-low-poly
description: Polybrush settings for TT low-poly terrain — Outer Radius 40, Sculpt Power 30, Falloff linear, Strength 1.0. Default settings produce sharp bumps incompatible with low-poly style.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/asset-pipeline
  tags: [unity, polybrush, terrain, settings, low-poly]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Polybrush Settings for Low-Poly Terrain

## Why settings matter

Default Polybrush settings produce sharp, spiky bumps — wrong for TT's low-poly aesthetic. TT terrain needs smooth, low-frequency undulation (gentle hills, shallow valleys). Settings must match the aesthetic before sculpting begins.

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
| Falloff | Soft (not linear) — for blended transitions between grass/dirt/rock |
| Strength | 0.4 |

## Post-Polybrush

Use Mesh → Smooth Vertex (custom Editor tool in TT) for additional softening of harsh transitions. Runs a Laplacian smooth pass over the mesh. Run 2-3 times on areas that still feel sharp.

## What "wrong" settings look like

Default Polybrush: Outer Radius 5, Power 80 → produces small sharp protrusions that look like inverted spikes. The low-poly style amplifies this — each vertex moves dramatically with small radius.

With tuned settings (Radius 40, Power 30): vertices move subtly across a wide area → smooth rolling hills that match TT's established terrain aesthetic.

## Save as preset

Polybrush supports brush presets (toolbar dropdown). Save these settings as "TT Low-Poly Sculpt" and "TT Low-Poly Smooth" to avoid re-entering per session.

See also: [[polybrush-iteration-rule]], [[backup-scene-before-modify]]
