---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, quest, highlight, ui, tutorial]
date: 2026-05-17
status: draft
---

# Quest Highlight Pattern (Quest-Flag Mechanism)

## When to use
Tutorial systems where specific objects need to be highlighted for exactly the duration of one quest, then unhighlighted permanently afterward. Prevents "always highlighted" objects in late game.

## Steps
1. Each highlightable object has `bool questHighlightActive` field
2. QuestManager.OnQuestStart(questId) → loop through objects tagged for that quest → set flag true → enable outline shader / emissive glow
3. QuestManager.OnQuestComplete(questId) → unset flag → outline disappears
4. Multi-step quests: highlight changes per step (object highlighted for its specific step only)

Highlight mechanism: outline shader on object mesh + animated emissive via MaterialPropertyBlock.

Critical: flag prevents re-highlight in later gameplay. A tree highlighted during "First Chop" tutorial is never highlighted again when player chops another tree of the same type.

## Why this works
Quest-flag is a gate: highlight is only active during the exact quest phase targeting that object. Removes tutorial context from gameplay context cleanly.

## Trade-offs
Requires objects to be marked as tutorial targets (Inspector or code). Worth it — alternative is permanent highlighting or complex state tracking.

## Variants
Same pattern for: NPC highlight during customer tutorial, machine highlight during "first use" quest, counter highlight during "repair counter" quest.

See also: [[multi-step-quest-checklist]] for quests with multiple highlight steps.
