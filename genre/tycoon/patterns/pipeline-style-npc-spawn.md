---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, npc, spawn, pipeline, event-driven]
date: 2026-05-17
status: draft
---

# Pipeline-Style NPC Spawn (OnPurchaseComplete Trigger)

## When to use
Tycoon games where customer flow should scale to player performance. Random/timed spawn = either empty parking lot or chaos. Pipeline spawn = steady flow matched to throughput.

## Steps

**NPCSpawner subscribes to purchase completion:**
```csharp
public class NPCSpawner : MonoBehaviour {
    [SerializeField] int maxConcurrentNPCs = 8;
    [SerializeField] float initialFillCount = 3;

    void OnEnable()  => onPurchaseComplete.Register(OnPurchaseComplete);
    void OnDisable() => onPurchaseComplete.Unregister(OnPurchaseComplete);

    void Start() {
        // Initial fill without purchase event
        for (int i = 0; i < initialFillCount; i++) SpawnNext();
    }

    void OnPurchaseComplete() {
        if (ActiveNPCCount < maxConcurrentNPCs) SpawnNext();
    }

    void SpawnNext() {
        var slot = ParkingManager.Instance.GetFreeSlot();
        if (slot == null) { QueueNextSpawn(); return; }
        var npc = npcPool.Get();
        npc.SetDestination(slot);
        npc.OnArrival += () => slot.Occupy(npc);
    }
}
```

**Free slot detection:**
```csharp
ParkingSlot GetFreeSlot() => slots.FirstOrDefault(s => s.occupant == null);
```

**Queued spawn:** if all slots occupied when next NPC should spawn, queue them. Fire queue when a slot becomes free.

**Spawn rate control (`spawnRate` parameter):**
- 3–5 NPCs/minute for active sawmill feeling
- At high player progression: higher max concurrent (scales with Reputation level)

**Initial fill on load** (see [[initial-fill-on-load]]): spawn N customers at game start without waiting for purchase events.

## Why this works
Pipeline spawn creates a feedback loop: sell more → customers replaced faster → opportunity to sell more. The loop naturally scales to the player's operational speed. No artificial timers that feel disconnected from gameplay.

## Trade-offs
- `maxConcurrentNPCs` cap: prevents infinite NPC accumulation if player is selling extremely fast. Tune to map capacity
- Queue overflow: if queue grows unbounded during high performance, cap queue size at 2× slot count
- NPC object pooling (see [[object-pooling-npcs-fifo-queue]]): essential — don't Instantiate/Destroy per spawn, reuse from pool

## Variants
- **Timed spawn:** ignore pipeline, just spawn one every N seconds. Simpler but feels disconnected from player activity
- **Demand-driven:** spawn NPCs based on inventory quantity (lots of planks → more customers wanting planks). More complex, higher engagement

See also: [[initial-fill-on-load]], [[object-pooling-npcs-fifo-queue]], [[npc-parking-pd-controller]]
