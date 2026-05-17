---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, state-machine, game-state, input-gating]
date: 2026-05-17
status: draft
---

# GameStateMachine Pattern

## When to use
Any game with more than one screen/mode (menu, gameplay, pause, cutscene). Without explicit state, input bleeds across states — pause menu lets player move, cutscenes get interrupted.

## Steps

**Enum and transitions:**
```csharp
public enum GameState { Boot, Splash, MainMenu, Loading, Playing, Paused, Cutscene }

// Transitions: Boot → Splash → MainMenu → Loading → Playing → (Paused | Cutscene) → Playing
```

**State entry effects:**
```csharp
void OnStateEnter(GameState newState) {
    switch (newState) {
        case GameState.Paused:
            Time.timeScale = 0f;
            // show pause UI
            break;
        case GameState.Cutscene:
            // disable player input — handled by input gate
            break;
        case GameState.Playing:
            Time.timeScale = 1f;
            break;
    }
}
```

**Input gating (in every system that reads input):**
```csharp
void Update() {
    if (GameStateMachine.Current != GameState.Playing) return;
    // process player input
}
```

**Finer gating with canMove flag** (combine with per-system flags for minigame states):
```csharp
// Minigame active: GameState = Playing but canMove = false
if (!playerController.canMove) return;
```

**ISaveable:** persist current state for crash recovery on next launch.

**States used in Timber Tycoon:**
- `Boot` — initial load, dependency init
- `Splash` — logo screen
- `MainMenu` — main menu
- `Loading` — async scene load
- `Playing` — full gameplay
- `Paused` — pause menu open, `Time.timeScale = 0`
- `Cutscene` — scripted sequences, player input blocked

## Why this works
State as a single shared enum means every system can check the same source of truth. `Time.timeScale = 0` at Paused stops all Update loops automatically, no manual "if paused, skip" needed.

## Trade-offs
- Monolithic enum: adding a new state requires editing the enum and all switch statements
- Nested states (minigame inside Playing): use per-system bool flags alongside GameState rather than adding enum values for every sub-state
- Cutscene camera: GameStateMachine controls input blocking, but camera routing to Cinemachine/sequence is a separate system concern

## Variants
- **Simpler version**: just `IsPlaying` bool + `IsPaused` bool for very small games
- **ScriptableObject-backed**: store current state in SO for easy Inspector debugging
- **State stack**: push/pop states (Paused → Playing recovery without Transition calls)

See also: [[parallel-architecture-pattern]], [[isaveable-contract]]
