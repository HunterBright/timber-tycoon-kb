---
name: initial-fill-on-load
description: NPC customers NOT serialized — on scene load, spawn N customers directly into free slots. Parking slot state IS saved (not NPC objects).
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, npc, save-load, initial-state, lifecycle]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Initial Fill on Load (Don't Serialize NPC State)

## When to use
Any tycoon/simulation game with transient NPC characters that move around the map. Serializing mid-walk NPC positions and AI states is complex, fragile, and bloats save files — avoid it.

## Steps

**What to save (ISaveable):**
```csharp
// ParkingSlotManager.GetSaveData()
{
    "slots": [
        { "id": "P00_W", "occupied": true, "customerType": "Regular" },
        { "id": "P01_E", "occupied": false },
        ...
    ]
}
```

**What NOT to save:**
- NPC position/rotation (mid-walk)
- NPC AI sub-state (driving to park, walking to counter)
- Transaction state (which items they're buying)

**On load — initial fill:**
```csharp
void LoadSaveData(string json) {
    var data = JsonUtility.FromJson<SlotSaveData>(json);
    foreach (var slotData in data.slots) {
        var slot = FindSlotById(slotData.id);
        if (slotData.occupied) {
            // Spawn NPC directly into slot (skip driving sequence)
            var npc = npcPool.Get();
            npc.transform.position = slot.transform.position;
            npc.transform.rotation = slot.transform.rotation;
            npc.SetState(NPCState.Parked, slot);
        }
    }
}
```

**Fill count on load:** scale to Reputation level. Rep Lvl 1 = 2 customers, Rep Lvl 5 = 5 customers, Rep Lvl 9+ = full parking lot.

**Edge case:** if all slots were empty at save → no initial fill. NPCs spawn naturally as gameplay resumes via pipeline spawn.

**Save format:** only IDs + occupied bool + customer tier. Not 3D positions. Tiny.

## Why this works
NPC state is inherently transient — a mid-walk NPC has no gameplay-critical state. The slot occupancy is the meaningful state (is a customer waiting to be served?). Reconstruct from that, not from saved 3D positions.

## Trade-offs
- Mid-transaction loss: if player saved during active sale, NPC transaction state is lost on reload — customer starts over. Acceptable (sale takes ~10s)
- Initial fill = "different customers" than before save: NPCs aren't the same instances. Players won't notice (all look similar, all have same behavior)
- Scaling fill count: if Reputation level changes between save and load (due to offline progression or cheat), fill count may differ from player's expectation

## Variants
- **Full NPC serialization:** save position + state + transaction per NPC. Complex, used in games like Cities: Skylines. Justified only if NPC identity matters to gameplay
- **No fill on load:** start with empty parking lot after load, let pipeline spawn fill naturally. Simpler, but initial few minutes feel empty

See also: [[pipeline-style-npc-spawn]], [[object-pooling-npcs-fifo-queue]]
