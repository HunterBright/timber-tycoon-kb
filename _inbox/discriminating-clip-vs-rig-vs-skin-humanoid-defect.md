---
title: Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-05-31'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mixamo
- humanoid-retargeting
- animation
- skinned-mesh
- diagnosis
- SampleAnimation
- x-bot
applies_to: []
source: ''
suggested-category: engine/patterns
name: discriminating-clip-vs-rig-vs-skin-humanoid-defect
---

# Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect

## Context
A Mixamo-rigged NPC played the shared Walk clip looking *wrong on one side* (left foot raised/twisted differently from the right; later, the whole mesh sitting at ~T-pose). Before touching anything, you must know which layer is at fault:
- **CLIP** — the shared animation (a central, batch-safe fix),
- **RIG/AVATAR** — the per-character skeleton/retarget calibration (a pipeline fix before batching N more bases), or
- **SKIN** — the mesh↔bone bind (a per-base mesh problem).

This entry keeps the discriminator method (genuinely reusable) **and** records how the metric misled us, so you don't repeat the chase. The real root cause this session turned out to be skin/bindpose corruption — see [FBX binary-overwrite corrupts bindposes](../lessons/fbx-binary-overwrite-corrupts-bindposes.md).

## The three-way discriminator (all read-only, no scene save)
1. **CLIP vs RIG** — apply the SAME clip to two different rigs and read **bone local rotations** (not the mesh). The rig-independent control is Mixamo's own **X Bot**: even when `X Bot@Walking.fbx` is animation-only (0 renderers, no mesh), the FBX still imports a valid avatar + skeleton GameObjects. Instantiate that meshless skeleton and sample onto it too.
   - `AnimationClip.SampleAnimation(go, t * clip.length)` at normalized `t = 0/0.25/0.5/0.75` (works in EDIT mode, no Play). Set `animator.runtimeAnimatorController = null` first so a controller can't overwrite the sampled pose. Read `animator.GetBoneTransform(HumanBodyBones.LeftFoot).localRotation.eulerAngles`, etc.
   - Healthy gait: `LeftFoot(t) ≈ RightFoot(t+0.5)` — X equal, Y&Z negated (mirror + half-cycle phase). If the reference rig is symmetric but the suspect rig shows a *constant* L/R offset at every phase, the clip is innocent.
2. **Decouple skeleton from mesh** — render the **skeleton as gizmos/debug lines** independent of the skin. This is the step that separates "bone motion" from "skin deformation": if the bones trace a correct, symmetric gait but the rendered mesh does not follow them, the fault is SKIN, not the bones or the clip.
3. **Check the binding** — `SkinnedMeshRenderer.bones[]`, `rootBone`, and `mesh.bindposes`. Counts are necessary but **not sufficient** (see "What the metric got wrong" below).

## What this ruled out (with evidence)
For NPC_Male2 the discriminator and follow-up tests cleared **every bone-level hypothesis**:
- **Not the clip** — the same clip on X Bot was symmetric.
- **Not bone→human mapping** — mirror-symmetric L vs R (~1–2°).
- **Not bind rotations / positions / bone lengths** — all mirror-symmetric.
- **Not bone roll** — a headless roll-fix (recompute roll so L = mirror of R) moved the metric **<0.5°**.
- **Not muscle axes** — the left/right limit-sign asymmetry read via reflection (`Avatar.GetPreRotation/GetPostRotation/GetLimitSign`) looked suspicious but was a red herring; correcting it did not move the visible defect.
- **Not the T-pose** — a genuine **Enforce T-Pose** via real `UnityEditor.AvatarSetupTool` (`MakePoseValid` / `TransferPoseToDescription`, reimport, `success = True`) moved the metric **<0.5°**.
- A **mathematically perfect mirror-symmetric skeleton still scored ~48°** on the toe metric.

The only thing that drove the foot metric down (to ~7.5°) was **Copy-From-Other-Avatar** (mapping the NPC through X Bot's avatar) — but it **visually wrecked the body**, because it retargets through X Bot's *proportions*. Rejected.

## The actual cause, and what the metric got wrong
The visible defect was **bindpose corruption** from binary-overwriting the FBX under a stale `.meta` — the mesh collapsed to ~T-pose while the bones animated correctly ([full entry](../lessons/fbx-binary-overwrite-corrupts-bindposes.md)). Every bone-space number read "fine" precisely because the bones *were* fine; the broken layer was the skin.

The "asymmetry" metric itself (toe-direction over 4 samples) was **over-sensitive** and partly misled the investigation: it reported high asymmetry on a skeleton that was, by construction, perfectly symmetric.

**Lasting lesson:** when a *clean, symmetric skeleton* scores high on an "asymmetry" metric, the **metric or a downstream step (the skin) is suspect — not the skeleton.** Don't keep "fixing" the skeleton. Decouple the skeleton from the mesh and compare what the bones do against what renders; trust the render. ([why](../lessons/debugging-search-first-trust-render-check-upstream.md))

## Gotchas
- Euler readouts near ±90° pitch (a planted/lifted foot) make Y/Z explode from gimbal — trust the pitch (X) delta, not absolute Y/Z magnitudes.
- Local-rotation NUMBERS are not comparable BETWEEN two rigs (different bind frames). Compare L-vs-R symmetry WITHIN each rig, then compare the two rigs' verdicts.
- The MCP scripting sandbox blocks `System.Reflection`; a compiled `Assets/Editor` script is not sandboxed and is the route to invoke real AvatarSetupTool / read internal Avatar axes.

## Related
- [FBX binary-overwrite corrupts bindposes (the real root cause here)](../lessons/fbx-binary-overwrite-corrupts-bindposes.md)
- [Character pipeline: Tripo → Mixamo → Unity (clean recipe)](../../workflow/asset-pipeline/character-pipeline-tripo-mixamo-unity.md)
- [Debugging methodology: search-first, trust the render, check for upstream sabotage](../lessons/debugging-search-first-trust-render-check-upstream.md)
