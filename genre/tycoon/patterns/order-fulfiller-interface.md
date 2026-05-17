---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, interface, order-fulfillment, player, npc, asymmetry]
date: 2026-05-17
status: draft
---

# OrderFulfiller Interface (Player vs NPC, Player Always Faster)

## When to use
Tycoon games where both the player and NPC workers can perform the same fulfillment task. Identical interface + asymmetric implementation = player always feels purposeful, automation feels like genuine assistance.

## Steps

**Interface definition:**
```csharp
public interface IOrderFulfiller {
    int CarryCapacity { get; }      // items per trip
    float MovementSpeed { get; }    // m/s
    float StorageDelay { get; }     // seconds pause at storage rack
    IEnumerator FulfillOrder(NPCCustomer customer);
}
```

**Player implementation:**
```csharp
public class PlayerController : MonoBehaviour, IOrderFulfiller {
    public int CarryCapacity => carryManager.CurrentCapacity;  // 1/3/5/8 by upgrade tier
    public float MovementSpeed => Input.GetKey(KeyCode.LeftShift) ? runSpeed : walkSpeed; // 5 or 10
    public float StorageDelay => 0f;  // immediate

    public IEnumerator FulfillOrder(NPCCustomer customer) {
        yield return GoToStorage(customer.productType);
        yield return WalkToCounter(customer);
        customer.ReceiveDelivery(carryManager.UnloadAll());
    }
}
```

**NPC worker implementation:**
```csharp
public class WorkerController : MonoBehaviour, IOrderFulfiller {
    public int CarryCapacity => 3;           // fixed, no upgrade path for this
    public float MovementSpeed => 3f;        // walk only, no sprint
    public float StorageDelay => 0.2f;       // brief pause while "searching" rack

    public IEnumerator FulfillOrder(NPCCustomer customer) {
        yield return new WaitForSeconds(StorageDelay); // NPC fumbles at rack
        yield return GoToStorage(customer.productType);
        yield return WalkToCounter(customer);
        customer.ReceiveDelivery(carryManager.UnloadAll());
    }
}
```

**Order queue:** `SalesCounter.orderQueue` accepts any `IOrderFulfiller`. Both player and NPC workers compete for queue assignments.

**Throughput comparison:**
- Player (Level 3, sprint): 8 items, 10 m/s, 0 delay → ~8× higher throughput than NPC
- NPC: 3 items, 3 m/s, 0.2s delay → consistent background throughput

**Late game:** player hires 2 NPC workers → 3 fulfillers active simultaneously. Player focuses on premium/VIP orders, NPCs handle regular volume.

## Why this works
Interface allows `SalesCounter` to work with any fulfiller without knowing if it's a player or NPC. `customer.ReceiveDelivery(fulfiller.CarryCapacity)` — same call, different capacity. Asymmetry is in the data, not the interface.

## Trade-offs
- Coroutine interface: `FulfillOrder` as `IEnumerator` requires `MonoBehaviour.StartCoroutine`. Can't call from pure C# classes — ensure all fulfillers are MonoBehaviours
- Sprint detection: `PlayerController.MovementSpeed` reads live input. NPC reads fixed value. Both implement the same getter correctly for their type.
- Future fulfillers: adding a "vehicle" fulfiller (truck delivers to wholesale customer) = implement `IOrderFulfiller` on `VehicleController`. Interface is extensible.

## Variants
- **Fully abstract orders:** `IOrderFulfiller.CanFulfill(Order)` returns bool — checks if fulfiller has inventory/capacity. Add before assigning orders.
- **Priority system:** VIP orders get assigned to player first, Regular orders go to NPCs. Implement as `OrderPriority` enum in the order queue.

See also: [[carry-capacity-progression-sprint]], [[worker-simulate-work-cycle]], [[customer-tier-system]]
