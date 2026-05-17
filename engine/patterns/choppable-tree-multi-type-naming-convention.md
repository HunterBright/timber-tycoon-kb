---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, prefabs, naming, scriptableobject, tree-types]
date: 2026-05-17
status: draft
---

# ChoppableTree Multi-Type Naming Convention

## When to use
Any data-driven system with multiple content variants (tree species, weapon types, vehicle models) that shares a single prefab and differentiates via ScriptableObject data.

## Steps
1. Define a string type field in the SO: `treeType = "Spruce"` (or use an enum)
2. Name all related assets: `{treeType}_{Suffix}` — strictly consistent
   - Prefabs: `Spruce_Stump`, `Spruce_Trunk_Fallen`, `Spruce_Log_1`, `Spruce_Sapling`, `Spruce_Small`, `Spruce_Medium`, `Spruce_Adult`
   - Materials: `Mat_Spruce_Trunk`, `Mat_Spruce_Stump`, `Mat_Spruce_Leaves`
3. `TreeTypeData` SO field list references these prefabs by name / Inspector assignment
4. Component reads SO at Start, applies prefab/material references (see [[scriptable-object-runtime-injection]])

Adding a new species:
1. Create `Assets/Models/Trees/{NewType}/` folder
2. Populate models following naming convention
3. Create `TreeType_{NewType}.asset` SO
4. Assign prefabs in Inspector
5. Done — zero code changes

## Why this works
Consistent naming enables scripted tools to find assets generically. More importantly: the naming pattern IS the documentation. `Maple_Log_2.prefab` is self-describing. `log_v4_final2.prefab` is not.

## Trade-offs
Requires naming discipline — one misnamed asset breaks the pattern's promise. Convention must be documented in CLAUDE.md / onboarding so every contributor follows it.

## Variants
Same pattern for: weapon types (`Axe_T1_Prefab`, `Axe_T1_Material`), NPC car variants (`NPCPickup01_Texture_Red`), machine tiers (`Chipper_T1`, `Chipper_T2`).
