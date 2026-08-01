---
title: 'In-Place FBX Overwrite: Safe for Static Meshes, Dangerous for Rigged'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-06'
project: Kerf - Sawmill Tycoon
tags:
- unity
- fbx
- reimport
- meta
- guid
- prefab
- fileIdsGeneration
- materials
- static-mesh
- rigged-mesh
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# In-Place FBX Overwrite: Safe for Static Meshes, Dangerous for Rigged

## Context
The rule "NEVER overwrite an FBX's bytes under its existing .meta" exists because for RIGGED/skinned meshes it collapses bindposes (T-pose / mesh explosion). But that rule is too broad if applied universally - for STATIC (non-skinned) props, overwriting the .fbx bytes in place while keeping the .meta is exactly the right way to re-import a corrected model without losing references.

## Validated this session (Pelletizer normal-fix re-import)
- Replaced `Assets/.../Pelletizer.fbx` bytes with a fixed-normals export; left the `.meta` untouched.
- Forced reimport: `AssetDatabase.ImportAsset(path, ForceUpdate | ForceSynchronousImport)`.
- Result: GUID unchanged (`6a456521…`); all scene/prefab references survived; 29 renderers re-imported clean; zero console errors.
- The prefab is a variant of the FBX with per-renderer material overrides (all 29 slots → one baked atlas material). After reimport **all 29 overrides were still intact (0 dropped)**.

## Why the prefab overrides survived
The model importer `.meta` had `fileIdsGeneration: 2` (deterministic file IDs derived from node names/types). Because the new FBX kept the SAME hierarchy and node names (only normals changed), the internal mesh/renderer fileIDs stayed stable, so the prefab modifications (which target those fileIDs) still matched and applied. If node names/hierarchy had changed, the overrides would have orphaned.

## Decision rule
- **Static / non-skinned mesh, same hierarchy + names** → overwrite in place under the existing .meta. Preserves GUID and every downstream reference (prefabs, scene instances, material remaps). Set/keep `fileIdsGeneration: 2` so prefab overrides survive.
- **Rigged / skinned mesh (has bones, Avatar, SkinnedMeshRenderer)** → do NOT overwrite in place; import as a NEW asset and re-wire. In-place overwrite collapses bindposes.

## Safety steps that paid off
1. Hash-compare source vs target first (confirm they actually differ and which is newer).
2. Git checkpoint commit of the current working tree BEFORE overwriting (recovery point).
3. After reimport, programmatically verify each renderer's `sharedMaterials` still points at the intended material; reassign + `PrefabUtility.SavePrefabAsset` ONLY if a slot dropped (non-destructive when nothing changed).

## Transferability
Any Blender→Unity re-export iteration loop on static props (machines, furniture, racks, environment pieces) where you want to keep the existing GUID and all references rather than re-link a fresh asset.

## Related
- [[fbx-export-standard-settings-blender-to-unity]]
- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[fbx-binary-overwrite-corrupts-bindposes|FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)]] - wspolne: meta, guid, fbx
- [[20260612-1340-unity-batch-fbx-import-meta-mirroring|Batch FBX import with pre-authored .meta files + prefab build in temp additive scene]] - wspolne: meta, prefab, fbx
- [[blender-mcp-interactive-remodel-loop|Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)]] - wspolne: guid, prefab, fbx
- [[20260606-1515-fbx-inplace-overwrite-fileid-preservation|Overwriting an FBX in place preserves prefab-variant references only if object NAMES are unchanged]] - wspolne: reimport, fbx
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] - wspolne: materials, fbx
- [[20260719-1605-spawn-pool-raw-fbx-bypasses-prefab|Anty-wzorzec: pula spawnera wskazuje surowy FBX zamiast prefabu-wrappera]] - wspolne: prefab, fbx
<!-- /POWIAZANE:auto -->
