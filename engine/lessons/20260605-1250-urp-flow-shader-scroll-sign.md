---
title: 'Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-05'
project: Kerf - Sawmill Tycoon
tags:
- shader
- urp
- water
- uv-scroll
- flow
- sign-convention
- vertex-displacement
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector

## Symptom
A custom flow/water shader (`Custom/LowPolyWater`) drove flow from a world-space
`_FlowDirection` (XZ) vector set at runtime by a `WaterFlow` MonoBehaviour. Setting the
vector to point "from source to mouth" made the water visibly flow BACKWARDS.

## Root cause
Both the UV-scroll and the vertex wave add `flowDir * _Time.y` to the sampled coordinate / phase:

```hlsl
uv      = pos.xz * scale + flowDir * _Time.y * _FlowSpeed;   // streaks
wavePhase = _Time.y * _WaveSpeed + dot(pos.xz, flowDir) * _WaveFrequency; // crests
```

For a feature of constant value, `pos·flowDir + t = const` ⟹ `pos` moves in **−flowDir**
as `t` increases. So the apparent motion (texture content AND wave crests) travels in the
direction OPPOSITE to the flow vector. To make water visually flow toward +X, set
`flowDirection = (-1, 0)`.

## Rule
When a shader adds `dir * time` to a UV or a sine phase, the on-screen motion is `−dir`.
Either negate the authored direction (as above) or subtract in the shader (`uv = pos - dir*t`)
so the inspector value matches the visible flow. Confirm by eye - sign bugs here read as
"flowing the wrong way," and it's a one-field flip.

## Related gotcha (same shader)
UV-based effects (e.g. `_SideWaveDamping` keyed off `uv` across the channel) silently
do nothing when the mesh has **no UVs** (`uvCount == 0`) - `uv` reads as 0 everywhere, so a
"bank damping" term collapses to a uniform amplitude scale. Verify the mesh actually has the
UV channel the shader assumes before tuning UV-driven parameters.

## Follow-up: a global flow vector flows SIDEWAYS on a curved channel
A single world-space flow vector only looks right where the channel happens to align with it.
On a meandering river it pushes water diagonally/across the channel on every bend. The fix is
to drive flow from a **channel-following UV channel** instead of world XYZ:

- Bake UVs onto the surface ribbon: **V = downstream arc-length along the centreline**
  (monotonic, 0 at source), **U = bank-to-bank 0..1 across the width**.
- Generating them without authored UVs: if the channel is monotonic along one world axis
  (verify: bin by that axis, check there's a single Z-cluster per bin - no folds), build the
  centreline from per-bin centroids, take cumulative segment length for V, and a local
  ±window min/max for U. (If it folds back, parameterise by an ordered centreline instead.)
- Shader then scrolls phase/UV along **V only** with a single scalar speed
  (`sin(t*speed − V*freq)` → crests travel toward +V). Flow now follows every bend, and the
  same U makes `_SideWaveDamping`/edge-fade localise to the banks as intended.
- Keep the old world-vector property declared-but-unused so the runtime setter component
  doesn't error; document that it no longer steers direction.
