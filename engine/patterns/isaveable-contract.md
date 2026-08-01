---
title: ISaveable Contract
type: pattern
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- save-system
- isaveable
- persistence
- json
applies_to: []
source: ''
suggested-category: engine/patterns
---

# ISaveable Contract

## When to use
Any Unity MonoBehaviour that needs to persist state across sessions. Implement ISaveable on every stateful component; SaveManager discovers and handles them automatically.

## Steps

**Interface definition:**
```csharp
public interface ISaveable {
    string GetSaveKey();              // unique per component, e.g. "VehicleStorage_Car1"
    string GetSaveData();             // serialize state to JSON string
    void LoadSaveData(string json);   // deserialize from JSON
}
```

**Minimal implementation:**
```csharp
public class EconomyManager : MonoBehaviour, ISaveable {
    float money;

    public string GetSaveKey() => "EconomyManager";
    public string GetSaveData() => JsonUtility.ToJson(new SaveData { money = this.money });
    public void LoadSaveData(string json) {
        var d = JsonUtility.FromJson<SaveData>(json);
        money = d.money;
    }
    [Serializable] class SaveData { public float money; }
}
```

**SaveManager auto-registration (Start):**
```csharp
void Start() {
    var saveables = FindObjectsOfType<MonoBehaviour>().OfType<ISaveable>();
    foreach (var s in saveables) Register(s);
}
```

**Save cycle:**
- Auto-save every 5 minutes
- 3 save slots
- SaveManager iterates registered ISaveables → builds combined key→JSON dict → writes to slot file
- Load: read file → dispatch each JSON blob to matching ISaveable by key

**Systems implementing ISaveable in Timber Tycoon:**
- `VehicleController` - position/rotation
- `VehicleStorage` - cargo list (ProductType × WoodSpecies)
- `QuestManager` - phase + completion flags
- `EconomyManager` - money
- `ToolManager` - unlocked tools + selected tool
- `AudioManager` - volume settings (5 floats)
- `WorldSpawnRegistry` - all runtime-spawned objects

## Why this works
Three methods = everything SaveManager needs. FindObjectsOfType scan at start means any new ISaveable added anywhere in the scene automatically participates in save/load without editing SaveManager.

## Trade-offs
- `FindObjectsOfType` scan at Start is O(all MonoBehaviours) - acceptable once at load, don't call per-frame
- Key uniqueness is developer responsibility - collision = one system overwrites another's data silently
- ISaveable on MonoBehaviours only - pure C# classes need a wrapper

## Variants
- **Awake-init guard for dependencies** (see [[awake-init-for-isaveable-with-dependencies]]): when ISaveable depends on another ISaveable, guard initialization so LoadSaveData runs after dependencies are ready
- **WorldSpawnRegistry extension**: dynamically spawned objects (GrowingTree, CollectableLog) register themselves in OnEnable/OnDestroy with WSR, which is the single ISaveable that saves/restores all of them

See also: [[parallel-architecture-pattern]], [[awake-init-for-isaveable-with-dependencies]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260611-2050-consumed-id-registry-save-pattern|Central consumed-ID registry for scene-object persistence]] - wspolne: persistence, save-system
- [[awake-init-for-isaveable-with-dependencies|Awake-Init for ISaveable with Dependencies]] - wspolne: isaveable, save-system
- [[20260716-0843-dual-owner-persistence-duplication|Dwoch wlascicieli trwalosci jednego obiektu w save (rejestr + rekonstrukcja stanu)]] - wspolne: persistence, save-system
- [[20260712-1820-save-migration-schema-version-gate|Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu]] - wspolne: isaveable, save-system
- [[20260622-1412-saveload-order-event-doublecount|Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu]] - wspolne: isaveable, save-system
- [[20260702-2200-save-system-missing-key-reset|Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)]] - wspolne: isaveable, save-system
<!-- /POWIAZANE:auto -->
