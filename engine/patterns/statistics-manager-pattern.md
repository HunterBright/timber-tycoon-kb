---
name: statistics-manager-pattern
description: StatisticsManager singleton — 11 lifetime stats + capped 200-event journal, ISaveable, UI with 2 tabs
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, statistics, journal, isaveable, ui, events]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# StatisticsManager Pattern

## When to use
Any game where players should be able to review their activity (playtime, production, economy) and recent events. Statistics fuel achievements, end-of-session summaries, and player retention. One component, hooks via existing events, near-zero ongoing cost.

## Steps

**StatisticsManager — tracked stats + event journal:**
```csharp
public class StatisticsManager : MonoBehaviour, ISaveable {
    // 11 lifetime counters
    public int treesCut, stumpsRemoved, saplingsPlanted;
    public int logsCollected, productsMade, customersServed;
    public float moneyEarned, moneySpent;
    public int daysPlayed, upgradesPurchased;
    public float kilometerssDriven;

    // Capped event journal
    const int MaxJournalEntries = 200;
    Queue<string> journal = new(MaxJournalEntries);

    public void AddJournalEntry(string entry) {
        if (journal.Count >= MaxJournalEntries) journal.Dequeue(); // drop oldest
        journal.Enqueue($"[Day {currentDay}] {entry}");
    }
}
```

**Hooks — increment from gameplay code via events:**
```csharp
// ChoppableTree raises OnTreeCut → StatisticsManager subscribes
void OnEnable() => onTreeCut.Register(OnTreeCut);
void OnTreeCut(TreeTypeData data) {
    treesCut++;
    AddJournalEntry($"Cut a {data.treeType} tree");
}
```

**11 stats tracked in TT:**
1. Trees cut (by species)
2. Stumps removed
3. Saplings planted
4. Logs collected
5. Products made (by type)
6. Customers served
7. Money earned
8. Money spent
9. Days played (TimeManager → OnDayChanged)
10. Kilometers driven (VehicleController distance accumulator)
11. Upgrades purchased

**UI:** Statistics panel with 2 tabs:
- **Stats:** scrollable list of all 11 stats, updated live
- **Journal:** scrollable list of last 200 events, newest at top

Access from PauseMenuUI → Statistics button.

**ISaveable:** all stats + journal entries serialized to JSON, restored on load.

**Achievement / end-of-demo hooks:** `OnTreesCutReached(milestone)`, `OnDayReached(day)` — raise via StatisticsManager for achievement triggers.

## Why this works
Statistics use already-existing GameEventSO channels — no new events needed. Queue with fixed max size prevents memory growth. Journal provides a "what did I do today" read for players.

## Trade-offs
- Journal cap at 200: at 1 entry per action, 200 entries covers ~10-20 minutes of play. Increase cap if journal is core UX
- Stats are global counters: no per-session or per-day breakdown unless you add session tracking
- String journal entries: lightweight but can't be filtered by type. Use an `EventRecord` struct (enum type + value) for filterable/categorized journal

## Variants
- **Achievement system:** StatisticsManager triggers achievements by monitoring counter thresholds — add `CheckAchievements()` call on each increment
- **Periodic summaries:** raise `OnDayChanged` → log "Day X summary: cut N trees, earned M gold" in journal automatically

See also: [[isaveable-contract]], [[game-event-so-event-channel]]
