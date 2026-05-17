---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, blender, naming, architecture, prefabs]
date: 2026-05-17
status: draft
---

# Architectural Elements Naming Convention

## When to use
Any building, structure, or machine hierarchy with multiple similarly-typed elements (beams, pillars, walls). Without consistent naming, hierarchy scanning and scripted setup require hardcoded index lookups or trial-and-error.

## Steps

**Naming scheme:**

| Element | Pattern | Examples |
|---------|---------|---------|
| Beams | `Beam_{Direction}_{Position}_Log` | `Beam_North_Log`, `Beam_East_Top_Log`, `Beam_Center_Log` |
| Pillars | `Pillar_{Compass}_{Height}` | `Pillar_NE_Tall`, `Pillar_NW_Short`, `Pillar_SE_Tall2` |
| Walls | `Wall_{Compass}` | `Wall_East`, `Wall_North`, `Wall_Triangle_North` (gable) |
| Foundation | `Foundation` | Single name, no variants |
| Roof | `Roof` or `Roof_{Dir}` | `Roof`, `Roof_North`, `Roof_South` (for split roofs) |

**Direction vocabulary:**
- Cardinal: `North`, `South`, `East`, `West`
- Composite compass: `NE`, `NW`, `SE`, `SW`
- Positional: `Top`, `Bottom`, `Center`, `Left`, `Right`

**Height variants:** `Tall`, `Short`, `Tall2` (for second identical pillar at same height)

**Gable walls:** `Wall_Triangle_{Compass}` distinguishes triangular gable ends from rectangular walls

**Scripted setup enabled:**
```csharp
// Find all pillars:
var pillars = GetComponentsInChildren<Transform>()
    .Where(t => t.name.StartsWith("Pillar_")).ToArray();

// Find north wall:
var northWall = transform.Find("Wall_North");
```

## Why this works
Predictable names = Inspector hierarchy is readable at a glance. Scripted setup scripts use `StartsWith("Beam_")` prefix matching — zero hardcoded indices. New elements follow the pattern = scripts find them automatically.

## Trade-offs
- No automatic enforcement: naming is a convention, not a rule. One off-name element = scripted setup misses it silently. Add an Editor validation script that warns on misnamed elements
- Blender names carry through to Unity (if FBX export includes object names): maintain naming in Blender, not just in Unity hierarchy
- `Tall2` suffix: hacky indicator for "second identical at same height." Consider `_A`/`_B` suffix or indexed `Pillar_NE_Tall_01`

## Variants
- **Indexed naming:** `Beam_01`, `Beam_02` — simpler but loses semantic meaning
- **Category prefix:** `B_BeamNorth`, `P_PillarNE` — useful when sorting by category in large hierarchies

See also: [[collider-distribution-rule]], [[choppable-tree-multi-type-naming-convention]]
