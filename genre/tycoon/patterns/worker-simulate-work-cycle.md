---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [game-design, tycoon, npc-worker, simulation, abstraction]
date: 2026-05-17
status: draft
---

# Worker Simulate Work Cycle (No NavMesh/AI)

## When to use
Tycoon games with automation workers (factory workers, farm hands, production station operators). At more than 3-4 workers, NavMesh + animation + pathfinding becomes too expensive and visually cluttered.

## Steps

**WorkerManager calls SimulateWorkCycle per assigned worker:**
```csharp
public class WorkerManager : MonoBehaviour {
    Dictionary<WorkerRole, List<WorkerInstance>> hiredWorkers = new();
    Dictionary<WorkerRole, float> cycleTimers = new();

    void Update() {
        foreach (var (role, workers) in hiredWorkers) {
            cycleTimers[role] += Time.deltaTime;
            float cycleTime = GetCycleTime(role); // baseWorkSpeed / speedMultiplier

            if (cycleTimers[role] >= cycleTime) {
                cycleTimers[role] = 0f;
                foreach (var worker in workers)
                    SimulateWorkCycle(worker);
            }
        }
    }

    void SimulateWorkCycle(WorkerInstance worker) {
        // 1. Check raw materials available
        int rawAvailable = StorageManager.Instance.Get(worker.GetRequiredInput());
        if (rawAvailable <= 0) { PlayIdleAnim(worker); return; }

        // 2. Consume inputs
        StorageManager.Instance.Remove(worker.GetRequiredInput(), worker.GetInputCount());

        // 3. Determine output quality
        var quality = worker.perfectQuality
            ? OutputQuality.Perfect
            : worker.GetOutputQuality(); // random distribution (see [[worker-output-quality-distribution]])

        // 4. Produce outputs
        int outputCount = GetOutputCount(quality);
        StorageManager.Instance.AddToStorage(worker.GetOutputType(), outputCount);
    }
}
```

**Visual representation:** static worker model at workstation. Idle animation plays when no materials. "Active" animation plays during cycle. No walking between stations.

**Player perception:** players understand abstraction — they're not watching 12 people walk, they're managing a business. "Worker is at the PlankMaker" is enough visual context.

**Performance:** 12 workers = 12 timer increments + 12 dictionary lookups per cycle. Negligible. 12 NavMeshAgents walking would cost 12 pathfinding calculations per frame.

## Why this works
Pure simulation: worker exists as data (WorkerInstance), not as scene object. No physics, no pathfinding, no collision. Production is a timer callback. Visual presence is cosmetic, not functional.

## Trade-offs
- Immersion: less "alive" than Stardew Valley workers who walk around. Acceptable trade-off for TT's scale (12+ workers). For 2-3 workers, walking animation might be worth it.
- Idle state: workers without materials show idle animation — players know production is blocked. Important feedback signal.
- Station-to-station transfer: worker can't "carry planks to the rack" in this model. Materials appear in storage instantly. Design around this: don't show workers carrying things; show worker at station, material appears in rack.

## Variants
- **Hybrid:** worker has "home" station + NavMesh walk animation between station and storage rack. Limited to ~3 workers before performance penalty. Use for hero workers (named characters, visible gameplay events).
- **Cinematic simulation:** workers only animate during a "production event" cutscene triggered every N cycles — boss-level workers feel more alive without constant animation cost.

See also: [[worker-data-instance-split]], [[worker-output-quality-distribution]], [[quantity-not-quality-design-principle]]
