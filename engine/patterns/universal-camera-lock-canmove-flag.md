---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, camera, minigame, input-gating, player-controller]
date: 2026-05-17
status: draft
---

# Universal Camera Lock — canMove Flag

## When to use
Any game with minigames, cutscenes, or interactive sequences where player movement and camera rotation should be blocked. Without a universal flag, each minigame reimplements its own camera disable logic — N implementations of the same thing, N places to forget the toggle.

## Steps

**PlayerController — single flag gates everything:**
```csharp
public class PlayerController : MonoBehaviour {
    public bool canMove = true;

    void Update() {
        if (!canMove) return; // blocks all movement AND camera rotation
        ProcessMovement();
        ProcessMouseLook();
    }
}
```

**Minigame start/end:**
```csharp
// Any minigame, anywhere:
playerController.canMove = false;  // lock
// ... minigame logic ...
playerController.canMove = true;   // unlock
```

**Combined with camera position lock** (for minigames that also move camera to a specific view):
```csharp
// canMove = false blocks rotation
// Camera position lock (see [[camera-lock-save-lerp-restore]]) handles position move
playerController.canMove = false;
cameraLock.LerpTo(minigameTarget, 0.5f); // move camera to machine view
```

**Combined with GameStateMachine** for double-safety:
- `canMove = false` prevents individual input processing
- `GameState.Paused/Cutscene` provides a scene-wide signal to systems that don't know about PlayerController
- Both checks for critical UI — neither alone is sufficient if some systems only check GameState

**New minigame checklist:**
1. Get ref to PlayerController
2. Set `canMove = false` on minigame start
3. Set `canMove = true` on minigame end (also in early-exit / ESC handler)
4. Done — no camera code, no input disabling code

## Why this works
Single flag, single location. Toggling `canMove` at the PlayerController level means all input processing for all axes stops simultaneously. No partial blocking (movement blocked but camera still spins) possible.

## Trade-offs
- `canMove` doesn't block UI input (inventory, pause menu can still open). That's intentional — if you need UI blocked too, use `GameStateMachine.SetState(GameState.Cutscene)`
- Null ref: if minigame code forgets to set `canMove = true` on error path, player is stuck. Always pair set-false with set-true in a finally block or coroutine cleanup
- Multiple lockers: if two minigames lock simultaneously (rare but possible), use a lock counter instead of bool: `lockCount++` / `lockCount--`, move only when `lockCount == 0`

## Variants
- **Lock counter:** `int moveLockCount = 0; bool canMove => moveLockCount == 0;` — handles overlapping locks safely
- **CursorLockMode:** combine with Unity's cursor lock to also hide/show cursor on minigame enter/exit

See also: [[camera-lock-save-lerp-restore]], [[game-state-machine-pattern]]
