---
name: zero-floating-zero-flickering-mandate
description: Hunter's mandate for ALL generated assets: ZERO visible gaps between adjacent surfaces, ZERO Z-fighting. Physical connection required. Check per checkpoint render before proceeding.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/3d-models
  tags: [blender, modeling, quality, z-fighting, mandate]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [blender-pipelines]
---

# ZERO Floating / ZERO Flickering Mandate

## The mandate

Every generated asset shipped to Unity must pass:
1. **ZERO floating elements** — no visible gap between adjacent surfaces
2. **ZERO Z-fighting flickering** — no coplanar surfaces at the same Z depth

## Failure modes

### Floating (visual gap)
Beam appears 0.05m above pillar top. Visible in-engine as a dark line/gap between surfaces.
- Cause: copy-pasted position values with rounding drift (`1.8` → `1.8000001`)
- Human eye threshold: gaps > 1mm visible at normal play distance

### Z-fighting (flickering)
Two faces occupy the exact same position in 3D space. GPU oscillates between rendering one or the other every frame = flickering, especially visible in motion.
- Cause: duplicate geometry, merged objects at exact same coordinates

## Fixes

**Adjacent surfaces that should TOUCH:**
```python
# WRONG — beam at Z=1.800001, pillar top at Z=1.8 → visible gap
beam.location.z = 1.800001

# CORRECT — beam and pillar share the same Z coordinate (they overlap by design)
PILLAR_TOP_Z = 1.800
beam.location.z = PILLAR_TOP_Z  # use shared constant
```

**Adjacent surfaces that should NOT overlap but be close:**
```python
# Intentional 1mm offset prevents Z-fighting without visible gap
surface_A.location.z = 0.5000
surface_B.location.z = 0.5010  # 1mm above surface_A
```

## Enforcement in scripts

Add assertions before save:
```python
# Assert Z alignment
pillar_top_z = pillar.location.z + pillar.dimensions.z / 2
beam_bottom_z = beam.location.z - beam.dimensions.z / 2
assert abs(pillar_top_z - beam_bottom_z) < 0.001, \
    f"Z mismatch: pillar={pillar_top_z:.5f}, beam={beam_bottom_z:.5f}"
```

## Per-checkpoint visual check

Each [[iterative-checkpoint-workflow-generated-assets]] render must include close-up inspection of all surface joints. If a gap or flicker appears in the preview → fix before proceeding to next checkpoint.

## Consequences

Floating gaps: player notices "looks cheap and unfinished." Z-fighting: player notices "this is glitchy." Both shipped = refunds and bad reviews. Mandate prevents.

See also: [[blender-headless-python-generation]], [[iterative-checkpoint-workflow-generated-assets]], [[procedural-textures-cycles-commercial]]
