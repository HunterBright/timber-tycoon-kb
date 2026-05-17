---
name: storage-rack-registry-auto-register
description: StorageRackRegistry singleton auto-registers racks via OnEnable/OnDisable — zero StorageManager edits when adding new racks
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, storage, registry, singleton, on-enable]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# StorageRackRegistry Singleton + Auto-Registration via OnEnable

## When to use
Any project with multiple storage receptacles of different types (inventory racks, chests, zones) where the manager should route to the correct rack without being manually wired. Works whenever a "registry" singleton can discover components by a category enum.

## Steps

**StorageRackRegistry singleton:**
```csharp
public class StorageRackRegistry : MonoBehaviour {
    public static StorageRackRegistry Instance { get; private set; }
    private Dictionary<StorageFamily, StorageRack> racks = new();

    void Awake() { Instance = this; }

    public void Register(StorageFamily family, StorageRack rack) => racks[family] = rack;
    public void Unregister(StorageFamily family) => racks.Remove(family);
    public bool TryGetRackByFamily(StorageFamily family, out StorageRack rack)
        => racks.TryGetValue(family, out rack);
}
```

**StorageRack self-registration:**
```csharp
public class StorageRack : MonoBehaviour {
    [SerializeField] public StorageFamily family;

    void OnEnable()  => StorageRackRegistry.Instance.Register(family, this);
    void OnDisable() => StorageRackRegistry.Instance.Unregister(family);
}
```

**Lookup from StorageManager:**
```csharp
if (StorageRackRegistry.Instance.TryGetRackByFamily(family, out var rack))
    rack.Add(product);
```

**Adding a new rack type:**
1. Drop GameObject with `StorageRack` component
2. Set `family` in Inspector
3. Done — zero StorageManager changes

**StorageFamily enum (Timber Tycoon):**
```csharp
public enum StorageFamily { Log, Firewood, Stump, Plank, Bark, ChipBag, PelletBag }
```

## Why this works
`OnEnable`/`OnDisable` fires automatically when the GameObject activates/deactivates or enters/exits the scene. The rack registers itself — no wiring ceremony. StorageManager looks up via enum key at call time, not at scene init.

## Trade-offs
- One family → one rack: first registered wins; if two racks share a family, only the latest survives. Extend to `Dictionary<StorageFamily, List<StorageRack>>` for overflow routing
- Execution order dependency: StorageRackRegistry.Awake must run before any StorageRack.OnEnable — enforce via Script Execution Order or hierarchy order
- No scene validation: if a rack family is expected but no rack exists, `TryGetRackByFamily` returns false silently — log a warning in StorageManager for debug builds

## Variants
- **Multi-rack per family** (capacity overflow): registry stores `List<StorageRack>`, StorageManager iterates until overflow is zero
- **General pattern**: any "service discoverable by type" uses the same OnEnable/Registry idiom — workstation machines, vendor zones, planting spots

See also: [[global-router-storage-pattern]], [[parallel-architecture-pattern]]
