---
name: iterative-checkpoint-workflow-generated-assets
description: Complex assets generated in checkpoints — partial asset → multi-angle preview → Hunter approval → next checkpoint. No jump-ahead. Error caught early = cheap. Caught late = redo everything.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, workflow, generated-assets, checkpoints, blender]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [claude-code-projects, blender-pipelines]
---

# Iterative Checkpoint Workflow for Generated Assets

## When to use
Complex 3D assets or multi-step generations where errors accumulate (Blender machines, characters, environment pieces). Anything where getting step 4 wrong means redoing steps 1-4. Use when: asset has 4+ distinct parts OR each part's correct execution depends on prior decisions.

## Steps

**Per checkpoint:**
1. Code generates partial asset (e.g., just the machine frame)
2. Renders multi-angle preview: isometric + top-down + front
3. Hunter reviews preview, responds: ✅ / 🚫 / 🔧
4. On ✅: proceed to next checkpoint
5. On 🔧: Code fixes, regenerates preview, back to step 3
6. On 🚫: rethink approach (rare — indicates wrong direction from start)

**Checkpoint sequence for a machine (PlankMaker example):**
1. Frame / base geometry
2. Primary functional component (bed, cutting head)
3. Secondary components (control panel, motor)
4. Details, buttons, labels
5. Final: texture assignment, UV check

## Why 4-6 checkpoints

Per-checkpoint cost: 5-15 min. Catching an error at checkpoint 2 costs 15 min to fix.

"Generate full machine in one shot" cost: if checkpoint 4's geometry is wrong, fixing it means redoing 2 hours of generation. The error isn't isolated.

**Bounded error:** one-shot generation = unbounded accumulated errors. Iterative = error per checkpoint ≤ one-checkpoint rework.

## ZERO floating mandate

At each checkpoint, verify the ZERO floating mandate (see [[zero-floating-origin-mandate]] if it exists): no mesh vertices, no empties, no objects floating in space with non-zero Y. Checkpoint is the moment to catch this, not after all geometry is assembled.

## Preview specification

```python
# Blender: capture 3 viewport angles
bpy.context.scene.camera.location = (5, -5, 4)  # isometric
render_to_file("checkpoint_isometric.png")
bpy.context.scene.camera.location = (0, 0, 10)  # top-down
render_to_file("checkpoint_top.png")
bpy.context.scene.camera.location = (0, -8, 2)  # front
render_to_file("checkpoint_front.png")
```

## Trade-offs

- Slower than one-shot if every checkpoint passes first try (rare for complex assets)
- Requires Hunter to be present at each checkpoint — can't run fully autonomously
- Worth it: reduces total rework time by 60-80% on complex assets

See also: [[quad-backtick-claude-code-prompt-format]], [[three-level-analysis-system]]
