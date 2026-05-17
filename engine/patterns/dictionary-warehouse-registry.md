---
name: dictionary-warehouse-registry
description: Dictionary<ProductType,int> warehouse inventory — zero physical GameObjects, O(1) lookup, auto-init on first access, visual layer separate
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, storage, dictionary, registry, data-driven]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Dictionary<ProductType, int> Warehouse Registry

## When to use
Any game inventory or warehouse system where the data model should be separated from the visual representation. Avoids "100 log GameObjects" scenarios that kill performance.

## Steps

**Core inventory in StorageManager:**
```csharp
public class StorageManager : MonoBehaviour, ISaveable {
    Dictionary<ProductType, int> inventory = new();

    public void Add(ProductType type, int amount) {
        inventory[type] = Get(type) + amount;
        OnInventoryChanged?.Invoke(type, Get(type));
    }

    public bool Remove(ProductType type, int amount) {
        int current = Get(type);
        if (current < amount) return false;
        inventory[type] = current - amount;
        OnInventoryChanged?.Invoke(type, Get(type));
        return true;
    }

    public int Get(ProductType type) {
        inventory.TryGetValue(type, out var count);
        return count; // returns 0 if absent — auto-init
    }
}
```

**Visual layer reads dictionary** — visual prefabs instantiated separately:
```csharp
// StorageRackVisual.cs subscribes to OnInventoryChanged:
void RefreshVisuals(ProductType type, int count) {
    // instantiate/destroy visual prefabs for this type
    // e.g., 1 visual stack per 10 logs
}
```

**ISaveable — serialize as JSON:**
```csharp
public string GetSaveData() {
    // Convert Dictionary to serializable struct
    var entries = inventory.Select(kv => new SaveEntry(kv.Key, kv.Value)).ToArray();
    return JsonUtility.ToJson(new SaveData { entries = entries });
}
public void LoadSaveData(string json) {
    var data = JsonUtility.FromJson<SaveData>(json);
    inventory.Clear();
    foreach (var e in data.entries) inventory[e.type] = e.count;
}
```

**Performance:** O(1) lookup via Dictionary key. No `GetComponentsInChildren` or scene iteration.

**Extension:** add `ProductType.NewItem` to enum → automatically tracked at 0 without any StorageManager code change.

## Why this works
Dictionary separates data from presentation. 1000 logs = 1 `int` in memory, not 1000 GameObjects. Visual layer can update independently at its own cadence (e.g., only on major fill changes, not every +1).

## Trade-offs
- No physical items in scene: some games need physics-based items (pickable stacks) — Dictionary model doesn't support that. Use a hybrid: Dictionary for inventory counts, separate `CollectableItem` pool for physics objects
- `ProductType` enum: adding values mid-project is safe (existing save files load fine — missing keys default to 0). Removing values breaks saved data that has that key
- Thread safety: Unity runs on a single thread for MonoBehaviours — no concurrent access concerns

See also: [[global-router-storage-pattern]], [[storage-rack-registry-auto-register]]
