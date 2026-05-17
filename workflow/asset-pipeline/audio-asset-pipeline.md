---
name: audio-asset-pipeline-elevenlabs-suno
description: Audio pipeline — ElevenLabs MCP for SFX + voice, Suno (manual) for music, FFmpeg normalize all to -16 LUFS. WAV for production, OGG for runtime. Naming conventions per channel.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: workflow/asset-pipeline
  tags: [elevenlabs, suno, ffmpeg, audio, pipeline]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects, claude-code-projects]
---

# Audio Asset Pipeline (ElevenLabs + Suno + FFmpeg)

## Tools

| Tool | Use | Access |
|------|-----|--------|
| ElevenLabs MCP | SFX generation, NPC voice bites | `mcp__elevenlabs__*` |
| Suno | Music generation | Web UI only — no MCP. Manual download. |
| FFmpeg | Normalize volume, convert format | CLI on local machine |

## Step-by-step

**1. Generate SFX via ElevenLabs MCP:**
```
mcp__elevenlabs__text_to_sound_effects({
  "prompt": "axe chopping through wood, mid-frequency thunk with brief wood crack resonance",
  "duration": 1.2
})
```
Output: `.wav` file

**2. Generate music via Suno (manual):**
- Web: suno.ai → describe track → download `.mp3`
- No automation available

**3. Normalize to -16 LUFS:**
```bash
ffmpeg -i input.wav -af "loudnorm=I=-16:LRA=11:TP=-1.5" normalized.wav
```
All audio in TT normalized to -16 LUFS. Consistent volume across all clips — no "this clip is way louder than others."

**4. Convert to OGG for runtime:**
```bash
ffmpeg -i normalized.wav -c:a libvorbis -q:a 4 final.ogg
```
OGG: smaller file size, Unity supports natively, good quality at q=4.

## File naming convention

| Channel | Folder | Naming |
|---------|--------|--------|
| SFX | `Assets/Audio/SFX/` | `sfx_kategoria_opis.ogg` |
| Music | `Assets/Audio/Music/` | `music_kategoria_opis.ogg` |
| Ambient | `Assets/Audio/Ambient/` | `ambient_strefa_opis.ogg` |
| UI sounds | `Assets/Audio/UI/` | `ui_akcja.ogg` |
| Voice | `Assets/Audio/Voice/` | `voice_npc_typ.ogg` |

Examples:
- `sfx_siekiera_cioswdrewno.ogg`
- `ambient_rzeka_loop.ogg`
- `ui_klikniecie.ogg`
- `voice_klient_zadowolony_01.ogg`

## AudioMixer routing

All clips assigned to mixer channels per [[audio-manager-mixer-architecture]]:
- SFX → SFX channel (default volume 0.8)
- Music → Music channel (default 0.4)
- Ambient → Ambient channel (default 0.6)

Player settings UI controls each channel independently.

## Why -16 LUFS

-16 LUFS is standard for games (Spotify/streaming standard is -14 LUFS, games slightly quieter to give headroom for sudden loud events). Unnormalized audio = random loud moments breaking immersion.

See also: [[audio-manager-mixer-architecture]], [[audio-strategy-minimal-music-heavy-ambient]], [[audio-mixer-snapshots-per-game-state]]
