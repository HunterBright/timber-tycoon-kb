---
type: anti-pattern
project: Timber Tycoon
suggested-category: engine/anti-patterns
tags: [unity, materials, textures, uv, baked-texture, atlas, blender, urp]
date: 2026-06-06
status: draft
---

# Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas

## The trap
Mesh B looks wrong, so you assign it Mesh A's nice baked material to "make it match" — e.g. giving a firewood rack frame the LogRack's `Mat_LogRack_Frame` (baked `LogRack_Frame_Diffuse.png`) so all racks share one wood look. Seems like a clean reuse.

## Why it fails
A **baked** texture (Cycles "bake selected to image") only paints color where the source mesh's UV islands sit; everything outside the islands is left as the **background (usually solid black, opaque)**. The baked texture is therefore valid ONLY for the exact UV layout it was baked from. Mesh B's UVs were laid out for a *different* texture, so its faces sample wherever B's islands happen to fall on A's atlas — frequently the **black background** → sharp-edged black patches (not shadows). Measured example: `LogRack_Frame_Diffuse` is ~16% black background; LogRack's own frame sampled 0/132 black faces, but the foreign firewood frame sampled **76/228 (33%) black faces**.

## Symptoms
- Sharp-edged black/dark blobs that follow the geometry (e.g. a post end goes fully black), unaffected by lighting/shadows.
- Other meshes using the SAME material look perfect (their UVs match the bake) — only the "borrowed" mesh is broken. This cross-check is the giveaway: shared texture is fine, the problem is per-mesh UVs.
- Not a material-slot leak (single slot), not flipped normals (0 inverted). Diagnostic: load the PNG from disk in an editor script, sample `tex.GetPixelBilinear(uvCentroid)` per triangle, count faces with luminance < ~0.12 — high count on one mesh, ~0 on the meshes the texture was baked for.

## Correct approach
Match the *tone*, not the baked atlas. Options, cheapest first:
1. **Solid-color material** (URP/Lit, `_BaseColor` only, no `_BaseMap`) — renders uniform regardless of UVs, zero black possible. Sample the source texture's island average color for the tone, store it in the material's linear `_BaseColor` (`avgSRGB.linear`, since GetPixels on an sRGB PNG returns gamma values and the URP albedo property is linear). Good enough when you only need a consistent color.
2. **Tileable (seamless) texture** that has no background — also UV-agnostic for 0–1 UVs.
3. **Re-bake** a texture for THIS mesh's own UV layout — only if you need matching baked grain/detail. This is texture work, NOT a mesh fix; never re-bake the shared atlas (it would break the meshes it currently serves correctly).

Rule of thumb: a baked atlas is bound to one UV layout. Reuse the *material concept* (tone/shader), not the baked pixels, across meshes with different unwraps.
