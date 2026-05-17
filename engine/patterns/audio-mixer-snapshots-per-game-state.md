---
name: audio-mixer-snapshots-per-game-state
description: Audio Mixer snapshots for clean game-state transitions — Paused mutes SFX, Menu boosts music, Playing restores full mix
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, audio, mixer, snapshots, game-state]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Audio Mixer Snapshots per Game State

## When to use
Any Unity game with multiple game states where audio mix should change — pause menu, cutscenes, main menu. Without snapshots, pause feels noisy and SFX bleed into UI screens.

## Steps

**Create snapshots in Audio Mixer asset:**
1. Right-click in Audio Mixer window → Add Snapshot
2. Create: `Playing`, `Paused`, `Menu`
3. Per snapshot, set mixer group volumes:

| Snapshot | Music | SFX | Ambient | UI |
|----------|-------|-----|---------|-----|
| Playing | 0 dB | 0 dB | 0 dB | 0 dB |
| Paused | -10 dB | -40 dB | -40 dB | 0 dB |
| Menu | +3 dB | -80 dB | -80 dB | 0 dB |

**Trigger from GameStateMachine:**
```csharp
// In AudioManager (or dedicated MixerStateManager):
void OnGameStateChanged(GameState newState) {
    string snapshotName = newState switch {
        GameState.Paused   => "Paused",
        GameState.MainMenu => "Menu",
        _                  => "Playing"
    };
    audioMixer.FindSnapshot(snapshotName).TransitionTo(0.3f); // 0.3s fade
}
```

**Hook to state machine:**
```csharp
void OnEnable()  => GameStateMachine.OnStateChanged.Register(OnGameStateChanged);
void OnDisable() => GameStateMachine.OnStateChanged.Unregister(OnGameStateChanged);
```

**Use cases:**
- Pause menu: SFX immediately dampened (machine hum, footsteps gone), UI clicks at full volume
- Cutscene: dramatic preset (music full, ambient down)
- Main menu: no gameplay audio, music prominent

## Why this works
Snapshots are an Audio Mixer feature — all transitions handled by Unity's audio engine with proper dB curves, no custom lerp code needed. Single `TransitionTo()` call changes the entire mix atomically.

## Trade-offs
- Setup cost: configuring N snapshots × M groups = N×M parameter values to tune
- Snapshot names are strings — typo-proof by using constants or enum-to-string conversion
- `TransitionTo(0f)` for instant cut (pause menu), `TransitionTo(0.3f)` for smooth crossfade (entering gameplay)

## Variants
- **Cutscene snapshot:** music + 3 dB, SFX -20 dB, ambient -10 dB — emphasis on score
- **Dialogue snapshot:** music -15 dB, ambient -15 dB, SFX -10 dB — player focuses on voice

See also: [[audio-manager-mixer-architecture]], [[game-state-machine-pattern]]
