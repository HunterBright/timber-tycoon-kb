---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, audio, audio-mixer, singleton, performance]
date: 2026-05-17
status: draft
---

# AudioManager + Mixer Architecture (5 Channels + 10-Source Pool)

## When to use
Any Unity game with more than 3 audio sources. Without a manager, AudioSources leak and audio categories (music vs. SFX) can't be controlled independently.

## Steps
1. Audio Mixer: Master → {Music, SFX, Ambient, UI} sub-groups
2. AudioManager (singleton, ISaveable): `Play(clip, AudioCategory)` routes to correct Mixer group
3. SFX pool: 10 pre-allocated AudioSources, rotate FIFO (interrupt oldest if all busy)
4. Music crossfade: 2 AudioSources alternating, 1s fade in / 1s fade out
5. Spatial blend: SFX/Ambient = 1.0 (3D positional), Music/UI = 0.0 (2D)
6. Volume saved as ISaveable: 5 floats (master, music, sfx, ambient, ui) at 0–1
7. Mixer snapshots per game state (see [[audio-mixer-snapshots-per-game-state]])

## Why this works
Single routing point = single volume control = single save. Pool of 10 AudioSources prevents GC spikes from Instantiate/Destroy. Mixer sub-groups = instant mute per category (pause menu, cutscene).

## Trade-offs
Setup cost: Audio Mixer asset config + 10 pre-allocated sources. One-time investment, infinite payoff.

## Variants
Simpler version: 4 AudioSources (music × 2 for crossfade + sfx_3D + ui_2D) for very small games.
