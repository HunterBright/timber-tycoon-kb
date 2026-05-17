---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, quest, ui, tutorial, multi-step]
date: 2026-05-17
status: draft
---

# Multi-Step Quest Checklist Pattern

## When to use
Tutorial or story quests with multiple sub-tasks where the player needs to know their exact progress. "Collect 5 items" without a visible counter = player frustration.

## Steps

**QuestSO structure:**
```csharp
[CreateAssetMenu(menuName = "Timber Tycoon/Quest")]
public class QuestSO : ScriptableObject {
    public string questName;
    public string questId;
    public List<QuestStep> steps;
    public bool stepsAreSequential; // false = any order
}

[Serializable]
public class QuestStep {
    public string description;          // "Collect 5 logs"
    public string completionCondition;  // event ID
    public int requiredCount;           // 0 = single completion, N = "collect N"
    [NonSerialized] public int currentCount;
    [NonSerialized] public bool isComplete;
}
```

**QuestUI rendering (checklist):**
```csharp
// Per step, render: [✓] or [☐] + description + counter if applicable
for (int i = 0; i < quest.steps.Count; i++) {
    var step = quest.steps[i];
    string status = step.isComplete ? "✓" : "☐";
    string counter = step.requiredCount > 1
        ? $" ({step.currentCount}/{step.requiredCount})"
        : "";
    stepTexts[i].text = $"{status} {step.description}{counter}";
}
```

**Real-time update via QuestManager.OnStepProgress:**
```csharp
QuestManager.Instance.OnStepProgress.Register(RefreshUI);
void RefreshUI(string questId, int stepIndex) {
    // Update specific step's counter and checkmark
}
```

**Quests in TT MVP (9 total):**

| Quest | Type | Steps |
|-------|------|-------|
| 1: CleanUp | Sequential | Collect 6 debris → repair counter |
| 2: GetAxe | Single | Pick up axe |
| 3: ChopTree | Sequential | Chop 1 tree → collect log |
| 5: Wycinka | Sequential | Chop 5 trees (each tree = 1 step) |
| 6: Sawmill | Sequential | Use sawmill once → place plank |
| ... | ... | ... |

**Sequential vs parallel:** `stepsAreSequential = true` — step N+1 grayed out until step N completes. `false` — all steps active simultaneously (e.g., "do A and B in any order").

## Why this works
Checklist UI makes progress tangible — player can count remaining boxes. Counter for "collect N" shows exactly how many more are needed. Visual completion checkmarks provide satisfying closure.

## Trade-offs
- Counter spoilers: showing "3/5" tells player exactly 2 more. For mystery quests, hide count. For tutorial, always show it.
- Step visibility: in complex quests, players may not read all steps initially. Consider "reveal next step on previous complete" for very long quests.
- Localization: step descriptions must be localization keys, not hardcoded strings.

## Variants
- **Objective tracker (GTA style):** always-visible top-right corner, one active quest only. Simpler, less detail.
- **Journal entries (Morrowind style):** rich text per step completion. More narrative, less visual progress indicator.

See also: [[quest-highlight-pattern]]
