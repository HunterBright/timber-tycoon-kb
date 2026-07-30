---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [blender, tripo, fbx, normals, mesh-cleanup, holes, unity, single-sided, ai-generated-mesh]
severity: high
time_lost: "~1 session of diagnosis across multiple renders"
date: 2026-05-31
status: draft
applies_to: [blender, unity, tripo, ai-mesh-pipeline]
---

# Tripo / AI-generated meshes import as "polygon soup" — see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces

## Problem
A Tripo-generated character mesh (5517 verts / 4812 tris) showed see-through holes in Unity (URP/Lit is single-sided / backface-culls). Symptom looked like "missing faces" or "interior faces poking through." Initial hypothesis was interior/overlapping geometry.

## Root cause
The mesh was effectively **polygon soup**: faces did NOT share vertices. `Merge by Distance` removed **3073 of 5517 verts** (>55%) — meaning almost every face had its own duplicate corner verts sitting on top of the neighbours'. A disconnected surface cannot have a consistent winding propagated across it, so `Recalculate Outside` (and Unity's own normal handling) leaves faces randomly flipped → flipped faces are culled → see-through holes. The actual "interior faces" count was only **2** (a near-no-op). The defect was the unwelded duplication, not interior geometry.

## Solution
Edit Mode cleanup IN THIS ORDER (order matters — recalc needs an as-connected-as-possible mesh):
1. **Merge by Distance** with threshold scaled to the model: `threshold = 0.0002 * max(mesh.dimensions)` (scale-independent regardless of FBX import unit). This reconnects the patches into one skin.
2. Select Interior Faces → delete (cheap, usually ~0 here, but harmless).
3. **Recalculate Outside** (`normals_make_consistent(inside=False)`) — only NOW can it orient all faces the same way, because step 1 connected them.

Verify with a render that MIRRORS Unity culling: SOLID shading + `space.shading.show_backface_culling = True` + a **magenta viewport background** (`background_type='VIEWPORT'`). Any remaining flipped/missing face shows as an unmistakable magenta patch straight through the surface. Render front AND back (a hole can face away from one camera).

## What didn't work
- Assuming "interior faces" was the culprit — it was 2 faces, essentially irrelevant.
- A normal double-sided render hides the defect entirely (both sides drawn) — you MUST cull backfaces to reproduce the Unity symptom.
- Trusting `Recalculate Outside` alone without merging first — on disconnected geometry it can't propagate.

## Transferability
Applies to ANY pipeline ingesting AI-generated / photogrammetry / marketplace meshes (Tripo, Hunyuan3D, Rodin, scanned assets) into Blender → Unity. The "merge-then-recalc, verify with backface-culled magenta render" recipe is engine-agnostic. The >50%-verts-merged signal is a reliable tell that you received polygon soup rather than clean topology.

## Related
- [[fbx-mesh-only-verification-scan-class-names]]
- [[humanoid-orientation-from-armature-not-bbox]]
