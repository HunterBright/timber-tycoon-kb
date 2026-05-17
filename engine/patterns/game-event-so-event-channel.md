---
name: game-event-so-event-channel
description: GameEventSO ScriptableObject event channel — Raise/Register/Unregister for decoupled messaging
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, scriptableobject, events, messaging, decoupled]
  severity: high
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# GameEventSO ScriptableObject Event Channel

## When to use
Any Unity project where two systems need to communicate without holding direct references to each other. Replaces `GetComponent`, `FindObjectOfType`, and event spaghetti with a clean ScriptableObject-based pub/sub channel.

## Steps

**Base class (generic + typed variants):**
```csharp
[CreateAssetMenu(menuName = "Timber Tycoon/Events/Game Event")]
public class GameEventSO : ScriptableObject {
    private List<Action> listeners = new();
    public void Register(Action listener)   => listeners.Add(listener);
    public void Unregister(Action listener) => listeners.Remove(listener);
    public void Raise() => listeners.ForEach(l => l.Invoke());
}

// Typed payload variant
public class GameEventSO<T> : ScriptableObject {
    private List<Action<T>> listeners = new();
    public void Register(Action<T> listener)   => listeners.Add(listener);
    public void Unregister(Action<T> listener) => listeners.Remove(listener);
    public void Raise(T data) => listeners.ForEach(l => l.Invoke(data));
}
```

**Subscriber pattern (mandatory — prevents listener leaks):**
```csharp
[SerializeField] GameEventSO onTreeCut;
void OnEnable()  => onTreeCut.Register(HandleTreeCut);
void OnDisable() => onTreeCut.Unregister(HandleTreeCut);
void HandleTreeCut() { /* respond */ }
```

**8 active channels in `Assets/ScriptableObjects/Events/`:**
- `OnMoneyChanged` — EconomyManager raises with new balance
- `OnTreeCut` — ChoppableTree raises on axe hit completion
- `OnLogPickup` — CollectableLog raises when player picks up
- `OnSale` — SalesCounter raises on customer purchase
- `OnDayChanged` — TimeManager raises at day transition
- `OnQuestComplete` — QuestManager raises with quest ID
- `OnUpgradeUnlocked` — SawmillManager raises with upgrade level
- `OnPlayerInteract` — PlayerInteraction raises on E-press

**Adding a new channel:** create `OnEventName.asset` in Events folder, wire in Inspector on both sender and receiver. Zero code changes to other systems.

## Why this works
Each channel is a ScriptableObject asset — assigned in the Inspector, not found at runtime. Sender raises, receivers register; neither knows about the other. Removing a system = just stop registering. Adding a system = just register on the channel.

## Trade-offs
- Listener leaks if Unregister is forgotten in OnDisable (use always-paired template above)
- Debugging: harder to trace "who raised this?" — use Unity's event debugger or add a debug log in Raise()
- Null payload: typed channels pass null on framework mistakes; add null guard in handler

## Variants
- **Parameterless** `GameEventSO` — most channels (state changes, triggers)
- **`GameEventSO<T>`** — when callers need data (OnTreeCut passes TreeTypeData, OnSale passes SaleAmount)
- VFX/audio hooks via `VFXTrigger.cs` / `AudioManager` subscribing to these channels (see [[vfx-trigger-pattern]])

See also: [[parallel-architecture-pattern]], [[isaveable-contract]], [[vfx-trigger-pattern]]
