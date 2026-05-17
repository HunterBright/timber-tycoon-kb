---
name: audio-strategy-minimal-music-heavy-ambient
description: Valheim-style audio strategy — minimal music (key moments only), heavy ambient (wind, footsteps, environment), ElevenLabs-generated voice bites in native language. Atmospheric > music-saturated.
metadata:
  type: decision
  project: eskimo-simulator
  suggested-category: genre/cross-genre/decisions
  tags: [game-design, audio, music, ambient, voice, cross-genre]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Audio Strategy — Minimal Music + Heavy Ambient + Voice Bites

## Decision

**Chosen:** Minimal music (key moments only) + rich ambient soundscape as primary audio + ElevenLabs-generated voice bites in native languages.

## Strategy breakdown

### Music (LOW priority — key moments only)
Inspired by Valheim: silence makes music land harder. Music only at:
- Main menu
- Major building completion (igloo finished, permanent shelter built)
- Dramatic encounter (polar bear, aurora event)
- Story-beat moments
- Victory / death screens

Between music moments: pure ambient. No looping background music.

### Ambient (HIGH priority — primary audio)
The ambient IS the soundtrack:

**Eskimo Simulator ambients:**
- Wind intensity (dynamic — scales with storm severity)
- Snow footstep layers (per depth: packed surface vs. deep powder)
- Cracking thin ice warning
- Qulliq (oil lamp) flame flicker
- Character breath in cold (distance from shelter metric)
- Wolves howling (time of day + player exposure)

**TT application:** sawmill ambient over music. Ambient river + birds + distant saw + wood creaking = more immersive than music loop.

### Voice (MEDIUM priority — authenticity layer)
ElevenLabs-generated. Use native languages for cultural authenticity:
- Eskimo Simulator: Inuktitut for Inuit NPC greetings/reactions
- Timber Tycoon: Polish customer exclamations (already in-region, reinforces setting)
- Short fragments only (greetings, emotional reactions, exclamations)
- Full dialogue = text with subtitles (AI voice for full sentences sounds worse than short phrases)

## Audio category mix (per [[audio-manager-mixer-architecture]])

| Channel | Eskimo Priority | TT Priority |
|---------|----------------|-------------|
| Music | Low | Low |
| Ambient | High | High |
| SFX | Medium | Medium |
| UI | Low | Low |
| Voice | Medium | Medium |

## Why ambient-priority wins over music-priority

Music-saturated games feel "video gamey." Ambient-priority games feel atmospheric and place-specific. If you're in an arctic blizzard, you should HEAR the blizzard — not hear an orchestral swell that happens to be playing.

The music that does exist hits harder because of the contrast. Valheim players remember the moment the combat music kicked in. That moment wouldn't land if combat music played constantly.

## Practical implementation note

**Don't write music first.** Write ambient sounds first. Fill the world with ambient detail. Then, in playtesting, notice which moments feel like they're "asking for music." Those are your 5-7 music moments. Write them last.

See also: [[audio-manager-mixer-architecture]], [[ambient-crossfade-zone-based]], [[audio-reverb-zone-per-environment]], [[footstep-raycast-surface-detection]]
