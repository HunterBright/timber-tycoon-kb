---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, vfx, game-event, decoupled, trigger]
date: 2026-05-17
status: draft
---

# VFX Trigger Pattern via GameEventSO

## When to use
Any game where gameplay systems (tree cutting, sales, machines) should trigger visual effects without directly referencing VFX prefabs. Adding new VFX should require zero changes to gameplay code.

## Steps

**VFXTrigger.cs component:**
```csharp
public class VFXTrigger : MonoBehaviour {
    [SerializeField] GameEventSO eventChannel;
    [SerializeField] GameObject vfxPrefab;
    [SerializeField] Vector3 offset;
    [SerializeField] float lifetime = 2f;

    void OnEnable()  => eventChannel.Register(OnEvent);
    void OnDisable() => eventChannel.Unregister(OnEvent);

    void OnEvent() {
        if (!VFXManager.Instance.RequestSpawn(/* particleCount */)) return;
        var go = Instantiate(vfxPrefab, transform.position + offset, Quaternion.identity);
        Destroy(go, lifetime);
    }
}
```

**Adding new VFX effect:**
1. Create VFX prefab in `Assets/VFX/`
2. Add `VFXTrigger` component to any GameObject in scene
3. Wire `eventChannel` → existing GameEventSO, `vfxPrefab` → new prefab
4. Done — no gameplay code changes

**Active VFX catalog (Timber Tycoon):**
| VFX | Event | Status |
|-----|-------|--------|
| Sparks | Sawmill active | Active |
| Smoke | Sawmill/Chipper active | Active |
| Golden coins | `OnSale` | Active |
| Vehicle exhaust | Vehicle moving | Active |
| Water splash | Vehicle enters river | Active |
| Sawdust (OnTreeCut) | — | WYCOFANE (too noisy) |
| Dust (OnTreeFall) | — | WYCOFANE |
| Leaves (OnTreeFall) | — | WYCOFANE |

**VFXManager budget check** (see [[vfx-performance-budget]]): VFXTrigger calls `VFXManager.Instance.RequestSpawn()` before spawning. If denied, effect silently skipped.

## Why this works
Gameplay code (ChoppableTree, EconomyManager) raises an event and forgets. VFXTrigger catches it and spawns. The connection is a ScriptableObject asset reference in the Inspector — no C# coupling. Removing VFX = delete the Trigger component.

## Trade-offs
- Position: VFXTrigger uses its own transform + offset. For world-position events (tree falls at specific spot), use `GameEventSO<Vector3>` typed channel and pass position as payload
- Multiple effects per event: add multiple VFXTrigger components, each wired to same channel
- Wycofane (withdrawn) effects: sawdust/dust/leaves were removed from TT as too visually noisy for low-poly aesthetic — document the decision, don't re-add without designer approval

## Variants
- **Audio + VFX combined**: `AudioTrigger.cs` follows same pattern, subscribed to same event channel — spawn sound + VFX simultaneously with zero coupling
- **Positional VFX**: typed channel `GameEventSO<VFXSpawnRequest>` with position + normal data

See also: [[game-event-so-event-channel]], [[vfx-performance-budget]]
