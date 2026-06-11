---
type: pattern
project: Timber_Tycoon
suggested-category: engine/patterns
tags: [unity, carry-system, viewmodel, camera, lifo, prefab, fpp, species-agnostic]
date: 2026-06-11
status: validated
---

# Stacked carry system — camera viewmodel + LIFO + species-agnostic prefab refs

## What it is
A first-person "carry a stack of items in view" system (logs on the shoulder, crates in hand) built entirely from existing prefabs, with no per-item code and no new save data.

## Key design points (all Play-Mode-verified)

1. **CarryRoot is a child of the PlayerCamera** — anchored on the right side of the view, exactly like the tool viewmodel (see [[tool-viewmodel-child-of-camera-pattern]]). The stack inherits camera sway/rotation for free and never clips through level geometry differently than tools do.
2. **Per-slot reference to the picked object's prefab.** Each carried slot stores a direct reference to the prefab of the thing that was picked up — not an enum, not a model lookup table. Any future species or item type works with **zero code changes** (see [[zero-code-changes-philosophy]]): pick it up, its own prefab is what gets displayed and what gets dropped.
3. **Longest-mesh-axis detection for orientation** — carried copies are oriented by programmatically detecting the mesh's longest axis from bounds, instead of assuming a local axis. This is what survives Blender FBX axis quirks across all 4 log species (see [[fbx-long-axis-detect-programmatically]]).
4. **Colliders and Rigidbodies are stripped on carried copies.** The viewmodel copies are purely visual — no physics fighting the player controller, no raycast interference with interaction prompts.
5. **Drop = top of stack (LIFO).** Dropping returns the most recently picked item; the visual stack and the data stack stay trivially in sync.
6. **Tool equip drops the whole stack.** Hands are exclusive: equipping a tool drops all carried items to the world. Avoids every "tool + carry simultaneously" edge case at the cost of one simple rule the player learns instantly.
7. **Save = drop-to-world.** On save, carried items are dropped as world objects, which the existing world-object persistence already handles — **no new save data structures** and no save-format migration.

## Why this shape
Every point trades a small visible constraint (LIFO order, drop-on-equip, drop-on-save) for the elimination of an entire class of code: no per-species visuals, no carry-state serialization, no physics special-casing. The system is additive on top of existing prefabs and the existing save pipeline.

## Planned reuse
Eskimo Simulator — same camera-viewmodel stack for carried fish/ice blocks; the species-agnostic prefab-ref slot design ports unchanged.

## See also
[[tool-viewmodel-child-of-camera-pattern]], [[zero-code-changes-philosophy]], [[fbx-long-axis-detect-programmatically]]
