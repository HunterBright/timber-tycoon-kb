---
name: do-not-move-hardcoded-positions
description: Scene objects with designer-tuned positions get // DO NOT MOVE comment. Code agents must respect this — never auto-adjust marked positions. Applied to cliff, waterfall, sawmill, bridge.
metadata:
  type: lesson
  project: timber-tycoon
  suggested-category: workflow/claude-code
  tags: [claude-code, scene, positions, convention, level-design]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects, claude-code-projects]
---

# "DO NOT MOVE" Hardcoded Positions Convention

## Rule

Scene objects positioned by a designer get a `// DO NOT MOVE` comment wherever their position is hardcoded in setup scripts.

```csharp
// DO NOT MOVE — manually positioned by Hunter, visual composition locked
Cliff_Waterfall.transform.position = new Vector3(184.5f, 7.2f, -82.3f);

// DO NOT MOVE — tartak position locked (all machine positions relative to this)
Sawmill.transform.position = new Vector3(177.9f, 7.62f, -88.71f);
```

## Scope in Timber Tycoon

Objects marked DO NOT MOVE:
- Cliff with waterfall
- Waterfall mesh
- Bridge
- River cave entrance
- Individual mountain positions (inner ring)
- Sawmill (177.9, 7.62, -88.71) — all machine layout depends on this
- Ramp/unload zone

Objects NOT marked (Code/Coplay can adjust):
- Enemy spawn points (future)
- NPC parking slots (designer expects auto-balancing)
- Debug objects, test cubes

## What Code agents must do

1. **When generating a RebuildFullMap.cs or similar:** include the DO NOT MOVE positions verbatim from the reference, don't calculate or offset
2. **When running Coplay `set_transform`:** before calling on any environment object, check if it has a DO NOT MOVE comment in associated scripts
3. **Never "optimize" a position** (e.g., "snapping cliff to terrain" = moves it off the designed composition)

## Why hardcoded, not data-driven

These objects were positioned by eye — the exact float coordinates are the design. The heightmap can't represent cliff position (terrain is flat under the cliff — it's a separate mesh floating). Each position was manually tuned for visual composition and gameplay layout. "Better" is not calculable.

## Common failure mode

"Code regenerated the map and now my cliff is in the river." Always happens because the regeneration script doesn't know which positions are sacred. The comment is the signal.

See also: [[backup-scene-before-modify]], [[scene-attachment-check-before-deleting-monobehaviour]]
