---
name: zero-code-changes-philosophy
description: Adding a new tree species (or product, worker role, NPC type) = create ScriptableObject + assign models = done. Zero code changes. Foundational TT philosophy for content scaling.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, content-scaling, philosophy, scriptableobject]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Zero-Code-Changes Philosophy

## Decision

**Chosen:** Any new content variant (species, product, worker role, vehicle type) = create a ScriptableObject + assign assets. Zero code changes required.

## New species workflow (reference implementation)

1. Create `Assets/Models/Trees/Oak/` — add `Oak_Adult.fbx`, `Oak_Stump.fbx`, `Oak_Trunk_Fallen.fbx`, `Oak_Log.fbx`, `Oak_Sapling.fbx`, growth stages
2. `Right-click → Create → Timber Tycoon → Tree Type Data → TreeType_Oak`
3. Assign models to `TreeType_Oak` fields in Inspector
4. Optionally: drop `TreeType_Oak` on existing tree prefab variant
5. Done — Spruce/Birch/Oak/Maple all coexist with zero code changes between them

## Systems that enable this philosophy

| System | Role |
|--------|------|
| [[scriptable-object-runtime-injection]] | TreeTypeData SO injected at runtime, not hard-coded |
| [[so-propagation-chain-via-parameters]] | Species flows through method params, not class state |
| [[storage-rack-family-system]] | Adding new family = enum value only |
| [[global-router-storage-pattern]] | Routes by enum key, not by type-specific code |

## Applied to

- Tree species (Spruce/Birch/Oak/Maple → Acacia/Mahogany zero-code)
- Product types (WoodSpecies enum)
- Worker roles (WorkerRole enum + WorkerData SO)
- Customer car variants (CarVariants SO + color pool)
- NPC customer types (NPCCustomerData SO)

## Trade-offs

- More SOs to maintain: each content variant has its own asset file. Worth it — asset management is cheaper than code management.
- Enum must be extended: StorageFamily, WoodSpecies, WorkerRole are enums. Adding a new value touches the enum file. That's unavoidable — but one line vs. multiple system changes.
- Test scope: new SO requires a smoke test (does it flow through the pipeline correctly?). Not code testing, but sanity testing.

## Implication for future projects

**Always** design content-variant systems with "zero code per variant" as the constraint. If adding a new variant requires code changes, the system isn't data-driven enough.

See also: [[scriptable-object-runtime-injection]], [[so-propagation-chain-via-parameters]], [[worker-data-instance-split]]
