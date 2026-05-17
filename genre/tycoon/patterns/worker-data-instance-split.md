---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, scriptableobject, worker, blueprint, runtime]
date: 2026-05-17
status: draft
---

# WorkerData (SO Blueprint) + WorkerInstance (Runtime) Split

## When to use
Games with hired worker/unit systems where workers can be upgraded. Upgrades affect runtime state (this specific worker), not the shared SO blueprint (would affect all workers of the same type).

## Steps

**WorkerData SO (shared blueprint, designer-tuned):**
```csharp
[CreateAssetMenu(menuName = "Timber Tycoon/Workers/Worker Data")]
public class WorkerData : ScriptableObject {
    public WorkerRole role;           // SalesCounter, Chipper, PlankMaker, etc.
    public float baseWorkSpeed;       // seconds per production cycle
    public int baseSalary;            // daily cost
    public string displayName;        // "Tartacznik Zuzanna"
    public Sprite portrait;
}
```

**WorkerInstance (runtime class — NOT MonoBehaviour):**
```csharp
[Serializable]
public class WorkerInstance {
    public WorkerData blueprint;              // reference to SO
    public bool isHired = false;
    public float workSpeedMultiplier = 1.0f; // 1.0 default, >1 = faster
    public bool perfectQuality = false;       // always Perfect output
    public string assignedStationId;          // which workstation

    public float ActualWorkSpeed => blueprint.baseWorkSpeed / workSpeedMultiplier;
}
```

**Worker upgrade (modifies instance, not SO):**
```csharp
// "Speed +20%" upgrade
worker.workSpeedMultiplier *= 1.2f; // compound: multiple speed upgrades stack

// Level 13 "Maestro Workforce" upgrade
worker.perfectQuality = true;
```

**WorkerManager.SaveData:** serialize `List<WorkerInstance>` per role. Each instance includes `blueprint.name` (to re-resolve SO ref on load) + runtime fields.

**Worker roles in TT:**
- SalesCounter — serves customers at counter
- Chipper — operates wood chipper
- PlankMaker — operates plank maker
- Pelletizer — operates pelletizer
- FertilizerMaker — (planned, Late Access)
- FurnitureWorkshop — (planned, Late Access)

## Why this works
SO blueprint is a shared reference — modifying it would change ALL workers of that type. Instance is per-worker state. Clear separation: designers edit blueprints, gameplay modifies instances.

## Trade-offs
- `WorkerInstance` as plain class (not MonoBehaviour): no Unity lifecycle (Awake, Update). `SimulateWorkCycle` called by WorkerManager on timer
- SO name resolution on load: use `blueprint.name` as save key, resolve via `Resources.Load` or SO reference list. Don't save GUID (fragile to asset moves)
- Salary: `baseSalary` per day. WorkerManager deducts from EconomyManager at each day transition (OnDayChanged event)

## Variants
- **WorkerInstance as MonoBehaviour:** place workers in scene as GameObjects. Simpler serialization, harder to scale past 5-6 workers.
- **Generic blueprint pattern:** same SO/instance split applies to buildings, upgrades, items — reuse the pattern across systems.

See also: [[worker-simulate-work-cycle]], [[worker-output-quality-distribution]], [[scriptable-object-runtime-injection]]
