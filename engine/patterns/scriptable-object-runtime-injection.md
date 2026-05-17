---
name: scriptable-object-runtime-injection
description: Component reads SO data in Start() with null-check fallback — zero code changes per new species/type, add via new SO + prefabs
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, scriptableobject, runtime-injection, extension-by-data]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ScriptableObject Runtime Injection Pattern

Merger of sweep2 "TreeTypeData runtime injection" + #108 (general SO injection pattern).

## When to use
Any component that needs to support multiple variants (tree species, weapon types, vehicle configs) where adding a new variant should require no code changes — only a new SO asset and prefabs.

## Steps

**Component has both SO field AND individual prefab fields:**
```csharp
public class ChoppableTree : MonoBehaviour {
    // SO-driven (preferred)
    [SerializeField] TreeTypeData treeTypeData;

    // Fallback Inspector fields (used when SO is null)
    [SerializeField] GameObject stumpPrefab;
    [SerializeField] GameObject fallenTrunkPrefab;
    [SerializeField] GameObject logPrefab;
    [SerializeField] TreeType treeType;

    void Start() {
        if (treeTypeData != null) {
            // SO-driven path: read all config from SO
            treeType = treeTypeData.treeType;
            stumpPrefab = treeTypeData.stumpPrefab;
            fallenTrunkPrefab = treeTypeData.fallenTrunkPrefab;
            logPrefab = treeTypeData.logPrefab;
            // etc.
        }
        // else: use Inspector-set values (fallback for legacy prefabs)
    }
}
```

**Concrete application — TreeTypeData SO:**
```csharp
[CreateAssetMenu(menuName = "Timber Tycoon/Trees/Tree Type Data")]
public class TreeTypeData : ScriptableObject {
    public TreeType treeType;           // Spruce, Birch, Oak, Maple
    public GameObject adultPrefab;
    public GameObject stumpPrefab;
    public GameObject fallenTrunkPrefab;
    public GameObject logPrefab;
    public GameObject saplingPrefab;
    public GameObject growingTreePrefab;
    public WoodSpecies woodSpecies;
    public Color barkColor;
}
```

**Adding new species (Oak → Mahogany):**
1. Create models in Blender (Adult, Stump, Trunk, Log, Sapling)
2. Create `TreeTypeData_Mahogany.asset` → assign all prefabs
3. Drop `ChoppableTree_Mahogany.prefab` in forest, assign SO
4. Done — zero code changes

**Same pattern applies to:**
- `WeaponData` SO for weapon variants
- `ItemData` SO for inventory items
- `VehicleData` SO for vehicle configs
- `MachineRecipeSO` for machine output variations

## Why this works
Content scaling. 4 tree species today, 12 tomorrow, same code. New species = data entry task, not engineering task. SO-driven path with Inspector fallback allows existing prefabs to keep working during migration.

## Trade-offs
- `Start()` injection: data is only available after Start() fires — don't read injected values in Awake()
- Race condition: if another script reads the component before Start() runs, it sees uninitialized values. Use `Init(TreeTypeData data)` method after Instantiate() for explicit initialization (see [[race-condition-start-vs-instantiate-parameter]])
- Null SO = fallback to Inspector fields: useful during development, but in production all prefabs should have a SO assigned

## Variants
- **Constructor injection:** `Init(TreeTypeData data)` method called after `Instantiate` — avoids Start() timing issue, cleaner for programmatically spawned objects
- **Generic SO base:** `TypeDataBase<T>` abstract SO class with common fields — reduces boilerplate across variants

See also: [[so-propagation-chain-via-parameters]], [[scriptable-object-runtime-assigned-references]], [[race-condition-start-vs-instantiate-parameter]]
