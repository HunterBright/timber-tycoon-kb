---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, scriptableobject, runtime, runtime-injection, extension-by-data]
date: 2026-05-17
status: draft
---

# ScriptableObject Runtime Injection Pattern

## When to use
Any component that needs to support multiple variants (tree species, weapon types, vehicle configs) where adding a new variant should require no code changes — only a new SO asset and prefabs. Avoids prefab explosion: single prefab + different SO data = multiple content variants.

## Component pattern (SO-driven with Inspector fallback)

Component has both SO field AND individual prefab fields:
```csharp
public class ChoppableTree : MonoBehaviour {
    [SerializeField] TreeTypeData treeTypeData;

    // Fallback Inspector fields (used when SO is null)
    [SerializeField] GameObject stumpPrefab;
    [SerializeField] GameObject fallenTrunkPrefab;
    [SerializeField] GameObject logPrefab;
    [SerializeField] TreeType treeType;

    void Start() {
        if (treeTypeData != null) {
            treeType = treeTypeData.treeType;
            stumpPrefab = treeTypeData.stumpPrefab;
            fallenTrunkPrefab = treeTypeData.fallenTrunkPrefab;
            logPrefab = treeTypeData.logPrefab;
        }
        // else: use Inspector-set values (fallback for legacy prefabs)
    }
}
```

## Spawner pattern (explicit SO passing)

Parent spawner passes SO after Instantiate, then calls explicit `Initialize()`:
```csharp
GameObject stump = Instantiate(stumpPrefab);
DiggableStump ds = stump.GetComponent<DiggableStump>();
ds.treeTypeData = this.treeTypeData; // pass from parent
ds.Initialize(); // explicit init after assignment — prevents race condition
```
Rule: NEVER use singleton lookup to find SOs — pass explicitly through the spawn chain.

## Concrete application — TreeTypeData SO

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

## Adding new species (zero code changes)

1. Create models in Blender (Adult, Stump, Trunk, Log, Sapling)
2. Create `TreeTypeData_Mahogany.asset` → assign all prefabs
3. Drop `ChoppableTree_Mahogany.prefab` in forest, assign SO
4. Done — zero code changes

Same pattern applies to: `WeaponData` SO, `ItemData` SO, `VehicleData` SO, `MachineRecipeSO`.

## Why this works

Content scaling. 4 tree species today, 12 tomorrow, same code. New species = data entry task, not engineering task. SO-driven path with Inspector fallback allows existing prefabs to keep working during migration.

## Trade-offs

- `Start()` injection: data only available after Start() — don't read injected values in Awake()
- Race condition: if another script reads the component before Start() runs, it sees uninitialized values. Use explicit `Init(TreeTypeData data)` after Instantiate() (see [[race-condition-start-vs-instantiate-parameter]])
- Null SO = fallback to Inspector fields: useful during development, but in production all prefabs should have a SO assigned

## Variants

- **Constructor injection:** `Init(TreeTypeData data)` called after `Instantiate` — avoids Start() timing issue, cleaner for programmatically spawned objects
- **Generic SO base:** `TypeDataBase<T>` abstract SO class with common fields — reduces boilerplate across variants

See also: [[so-propagation-chain-via-parameters]], [[race-condition-start-vs-instantiate-parameter]]
