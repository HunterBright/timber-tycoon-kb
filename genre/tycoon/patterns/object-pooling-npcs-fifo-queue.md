---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, npc, object-pool, performance, fifo]
date: 2026-05-17
status: draft
---

# Object Pooling for NPCs + FIFO Customer Queue

## When to use
Any game where NPCs are frequently spawned and despawned (customers arriving/leaving, enemies respawning). `Instantiate/Destroy` per NPC at 100 NPCs/hour = GC churn, frame spikes.

## Steps

**Simple NPC pool:**
```csharp
public class NPCPool : MonoBehaviour {
    [SerializeField] GameObject npcPrefab;
    [SerializeField] int poolSize = 8;
    Queue<NPCController> available = new();

    void Start() {
        for (int i = 0; i < poolSize; i++) {
            var npc = Instantiate(npcPrefab);
            npc.SetActive(false);
            available.Enqueue(npc.GetComponent<NPCController>());
        }
    }

    public NPCController Get() {
        if (available.Count == 0) return null; // pool exhausted, deny
        var npc = available.Dequeue();
        npc.gameObject.SetActive(true);
        return npc;
    }

    public void Return(NPCController npc) {
        npc.ResetState();
        npc.gameObject.SetActive(false);
        available.Enqueue(npc);
    }
}
```

**FIFO customer queue at SalesCounter:**
```csharp
public class SalesCounter : MonoBehaviour {
    Queue<NPCCustomer> waitingQueue = new();

    void OnCustomerArrives(NPCCustomer customer) => waitingQueue.Enqueue(customer);

    void ServeNext() {
        if (waitingQueue.Count == 0) return;
        var customer = waitingQueue.Dequeue();
        StartTransaction(customer);
    }

    void OnTransactionComplete(NPCCustomer customer) {
        customer.Depart(); // walks to car → drives out → NPCPool.Return()
        ServeNext();
    }
}
```

**Pool size rationale:**
- 8 NPCs: at 3-5 customers/min, 8 = enough for ~2 min of queue buildup
- Scales with progression: `poolSize = Mathf.Clamp(reputation / 10 + 3, 3, 12)`
- Pre-instantiate at `Start()` so no GC during gameplay

**NPC.ResetState():** clears position, animation state, transaction data, destination before returning to pool.

## Why this works
Pre-allocated pool = zero Instantiate/Destroy calls at runtime. All memory is allocated upfront. FIFO queue ensures customers are served in arrival order (fair) and counter never serves 2 customers simultaneously.

## Trade-offs
- Pool exhausted: `Get()` returns null → spawner skips that spawn. Acceptable — better than unlimited allocation. Log a warning in debug builds
- Recycle bug: if `ResetState()` doesn't clear all state, recycled NPC carries ghost data from previous use. Always write an explicit reset method
- Pool size constant: at high Reputation, 8 may not be enough. Make it a runtime-adjustable parameter

## Variants
- **Unity ObjectPool<T>** (since Unity 2021): built-in pool with event callbacks. Prefer this for new projects
- **Separate pools per NPC type:** `regularPool`, `contractorPool`, `vipPool` — more memory, zero type confusion

See also: [[pipeline-style-npc-spawn]], [[initial-fill-on-load]]
