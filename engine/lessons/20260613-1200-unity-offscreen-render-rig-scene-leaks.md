---
title: An "isolated" offscreen render rig still inherits the open scene's lights AND skybox reflection
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- urp
- render-to-texture
- icon-baking
- thumbnail
- lighting
- determinism
- reflection-probe
- culling-mask
applies_to: []
source: ''
promoted: '2026-07-30'
---

# An "isolated" offscreen render rig still inherits the open scene's lights AND skybox reflection

## Context
Baking item icons from 3D models with an in-editor RenderTexture rig (ortho camera + own
directional lights, subject moved far away onto a dedicated layer 30, camera cullingMask =
that layer only). Goal: deterministic icons that look the same regardless of which scene is
open. The rig looked isolated but was not.

## Symptom
- Tuning the rig's own lights (key/fill/rim intensities & colors) barely changed the output -
  bright materials clipped to white no matter what.
- A warm-brown firewood model rendered cold blue-grey on its broad faces, while its end-grain
  stayed warm. Warming every rig light did almost nothing.

## Root causes (TWO separate leaks)
1. **Directional light leak.** Putting the subject on layer 30 and masking the *camera* to
   layer 30 does NOT stop the SCENE's lights from lighting it. A Light affects an object when
   `light.cullingMask` includes the object's layer; scene Suns default to `Everything`, so they
   lit layer 30 and dominated the rig. Proof: at rig key intensity 0.40 the brightest face was
   still ~1.0 (impossible from 0.40 alone).
2. **Environment specular leak.** Smooth materials reflect `RenderSettings` ambient/reflection.
   `ambientMode = Flat` overrides ambient color but NOT the reflection source - the scene's
   skybox/reflection probe still feeds specular, so smooth faces reflected the scene's blue sky
   and read cool. This is why warming the lights didn't help: the blue was a reflection.

## Fix (save → set → restore around the render)
- For every active scene `Light`, clear the render layer from its `cullingMask`
  (`l.cullingMask &= ~(1 << RENDER_LAYER)`), skipping the rig's own lights. Restore after.
- `RenderSettings.reflectionIntensity = 0` for the render (kills skybox/probe specular). Restore after.
- Also set deterministic `ambientMode = Flat` + low flat `ambientLight`, and `fog = false`
  (save/restore all). Never save the scene - restore everything in memory.

## Generalizable rule
A render-to-texture rig is only isolated for GEOMETRY (camera cullingMask), not for LIGHTING.
Full lighting isolation needs THREE things saved/restored: (a) scene lights' per-layer culling,
(b) ambient (mode + color + intensity), (c) `reflectionIntensity` (+ fog). Miss any one and your
"deterministic" bake silently depends on whatever scene happens to be open.

## Bonus
- AA on an offscreen RT: supersample (render at N× then box-downsample) is more reliable than
  fighting MSAA-resolve-on-readback under SRP; it's pipeline-agnostic and guarantees clean edges.
- Framing many shapes consistently: fit the ortho size to the subject's PROJECTED on-screen
  bounds (camera-space min/max of the 8 AABB corners), one shared padding - not the 3D
  bounding-sphere diagonal (which makes elongated items render tiny).
