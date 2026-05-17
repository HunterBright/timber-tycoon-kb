---
name: planting-spot-universal-not-typed
description: PlantingSpot is species-neutral. Original design typed spots (only same species can be replanted). Revised to any sapling in any spot. More fun, enables biome-shift gameplay.
metadata:
  type: decision
  project: timber-tycoon
  suggested-category: genre/tycoon/decisions
  tags: [game-design, planting, tree, decision-record]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# PlantingSpot Universal (Not Typed by Species)

## Decision

**Chosen:** PlantingSpot is species-neutral. Any sapling can be planted in any spot. The sapling carries the species information.

## Options considered

| Option | Description | Verdict |
|--------|-------------|---------|
| Typed spot | PlantingSpot remembers what was cut, only same species replanted | REJECTED |
| Universal spot | Any sapling, any spot. Sapling carries treeTypeData | SELECTED |

## Why universal wins

**Typed spots (rejected):**
- "Realistic" — a spruce stump can only grow a spruce back
- But: forces player to carry specific sapling types to specific spots
- Consequence: player spends time matching saplings to spots instead of playing the tycoon loop
- Worse: if player only has Birch saplings but only Spruce stumps are available, progression is blocked

**Universal spots:**
- Player picks up any sapling, plants anywhere
- Enables **biome-shift gameplay**: "I'm converting this boring Spruce forest into high-value Maple trees"
- Creates interesting long-term choices: which species to grow where?
- No progression blocking

## Implementation

```csharp
// PlantingSpot — no species field
public class PlantingSpot : MonoBehaviour {
    // treeTypeData is NOT stored here

    public void PlantSapling(PickupSapling sapling) {
        var growingTreePrefab = sapling.treeTypeData.growingTreePrefab;
        // Instantiate at this spot position — sapling type determines what grows
    }
}
```

The `PickupSapling` carries `TreeTypeData`. `PlantingSpot` reads it at plant time. After planting, `GrowingTree` grows into the correct adult species.

## Lesson

Design constraints should serve gameplay, not realism. "It would be realistic for only spruce to grow in a spruce spot" is a weak argument if it makes the game less fun. Iterate based on what creates interesting player choices, not biological accuracy.

See also: [[scriptable-object-runtime-injection]], [[zero-code-changes-philosophy]], [[so-propagation-chain-via-parameters]]
