---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, audio, reverb, environment, ambient]
date: 2026-05-17
status: draft
---

# AudioReverbZone per Environment

## When to use
Any game where the player moves between acoustically distinct spaces (building interior, outdoor forest, cave, bridge over water). Same footstep sound in every space breaks the sense of place.

## Steps

**Unity's built-in AudioReverbZone component:**
- Place `AudioReverbZone` on a GameObject at each environment boundary
- Set `Min Distance` (full reverb effect) and `Max Distance` (reverb fades to zero)
- Listener moves through zone → reverb applied to all AudioSources

**Preset assignment by environment:**

| Environment | Preset | Notes |
|-------------|--------|-------|
| Sawmill interior | SmallRoom | Low ceiling, wooden walls |
| Forest | Forest | Light scatter, no echo |
| Open field | Off | Dry, no reverb |
| Bridge over river | Concerthall (low decay) | Water surface reflection |
| River cave | Cave | Long decay, high density |

**Overlap zones** at transitions (e.g., doorway): inner zone = indoor, outer zone = outdoor, overlapping region = blended reverb. Player transitions seamlessly.

**Setup:**
1. Create empty GameObject at zone center
2. Add AudioReverbZone → set preset → set Min/Max Distance
3. Tune: stand in zone in Play Mode, listen, adjust distances

**Zones in Timber Tycoon:**
- Sawmill interior (SmallRoom)
- Forest clearing (Forest preset)
- Covered bridge (Concerthall)
- River cave (Cave)
- Open terrain (no zone = dry default)

## Why this works
AudioReverbZone is Unity built-in, zero code needed. Listener location determines reverb automatically. Overlap = automatic smooth blending between zones at no extra cost.

## Trade-offs
- Up to 4 `AudioReverbZone` components can affect the listener simultaneously — don't place too many overlapping
- Presets are approximate — tune Min/Max Distance so that zones don't bleed into inappropriate spaces
- Doesn't work well with 2D audio (UI, music) — those bypass spatial processing anyway

## Variants
- **Script-driven reverb snapshots**: trigger custom reverb presets from `AmbientZoneSO` (more control, more code)
- **Occlusion combination**: pair with `AudioOcclusionChecker` (see [[audio-occlusion-lpf-volume]]) for full spatial audio treatment

See also: [[ambient-crossfade-zone-based]], [[footstep-raycast-surface-detection]]
