---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, scriptableobject, progression, data-driven, reputation]
date: 2026-05-17
status: draft
---

# ReputationLevels.asset Data-Driven Progression

## When to use
Any progression system with discrete thresholds and unlock gates where the number and values of levels will be tuned during balancing. Hard-coded levels = a code change per balance pass.

## Steps

**ReputationLevelsSO:**
```csharp
[CreateAssetMenu(menuName = "Timber Tycoon/Reputation/Levels Config")]
public class ReputationLevelsSO : ScriptableObject {
    public List<ReputationLevel> levels;
}

[Serializable]
public class ReputationLevel {
    public int threshold;           // rep points required to reach this level
    public string nameKey;          // localization key, e.g. "Reputation.Level.Drwal"
    public List<string> unlocks;    // upgrade IDs unlocked at this level
}
```

**ReputationManager.GetCurrentLevel():**
```csharp
public int GetCurrentLevel() {
    int level = 0;
    for (int i = 0; i < levelsConfig.levels.Count; i++) {
        if (currentRep >= levelsConfig.levels[i].threshold)
            level = i;
        else break;
    }
    return level;
}

public ReputationLevel GetLevel(int index) => levelsConfig.levels[index];
```

**Level names (localized) in Timber Tycoon:**

| Level | Polish name | Rep threshold |
|-------|-------------|---------------|
| 0 | Nowicjusz | 0 |
| 1 | Drwal | 100 |
| 2 | Tartacznik | 250 |
| 3 | Mistrz Tartaku | 500 |
| ... | ... | ... |
| 12 | Legenda Lasu | 5000 |

**Scaling story:** Initial design: 5 levels (MVP). Scaled to 13 during balancing — editing the `ReputationLevels.asset` Inspector, zero code changes.

**Localization:** level names use localization keys. `LocalizationManager.Get("Reputation.Level.Drwal")` returns language-appropriate string.

## Why this works
All level data lives in one place (the SO). Balance team changes thresholds without touching code. New levels = add entries to list. Remove levels = delete entries. Code reads dynamically, handles any list length.

## Trade-offs
- Level count driven by data: if `levels.Count == 0`, `GetCurrentLevel()` always returns 0 — add a validation check
- `unlocks` list as strings: typo-proof via using the same string constants as the upgrade system. Consider an `UnlockKey` enum instead of raw strings
- Localization key coupling: level name keys must exist in localization files. Missing key = displayed as raw key in UI — add Editor validation to check all level name keys exist

## Variants
- **JSON config:** same data in an external JSON file, loaded at runtime — allows patches without rebuilding. More complex, useful for live-service games
- **Inline thresholds:** thresholds defined in ReputationManager code as `static readonly int[]` — faster iteration for programmers, worse for designers

See also: [[scriptable-object-runtime-injection]], [[migration-pattern-rollback-safety]]
