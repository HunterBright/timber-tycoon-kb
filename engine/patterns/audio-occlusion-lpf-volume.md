---
name: audio-occlusion-lpf-volume
description: Cheap audio occlusion via raycast — obstructed sources get Low Pass Filter + volume reduction, 0.1s update interval
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, audio, occlusion, low-pass-filter, spatial]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Audio Occlusion Pattern (LPF + Volume)

## When to use
3D spatial AudioSources that should sound different from behind a wall vs. in open air. Machine sounds, ambient interior hum, NPC voices — all should be muffled when the listener is blocked from the source.

## Steps

**Per-AudioSource occlusion component:**
```csharp
public class AudioOcclusionChecker : MonoBehaviour {
    AudioSource source;
    AudioLowPassFilter lpf;
    float checkInterval = 0.1f;
    float timer;

    void Start() {
        source = GetComponent<AudioSource>();
        lpf = gameObject.AddComponent<AudioLowPassFilter>();
        lpf.enabled = false;
    }

    void Update() {
        timer += Time.deltaTime;
        if (timer < checkInterval) return;
        timer = 0f;
        CheckOcclusion();
    }

    void CheckOcclusion() {
        var listener = Camera.main.transform.position;
        var dir = (listener - transform.position).normalized;
        bool occluded = Physics.Raycast(transform.position, dir,
            Vector3.Distance(listener, transform.position),
            obstructionLayerMask);

        lpf.enabled = occluded;
        lpf.cutoffFrequency = occluded ? 800f : 22000f;
        source.volume = occluded ? baseVolume * 0.5f : baseVolume;
    }
}
```

**Key values:**
- `cutoffFrequency = 800 Hz` when occluded (muffled behind concrete)
- Volume multiplier: 0.5 (reduce ~50% when blocked)
- Update interval: 0.1s — sufficient for audio, avoids per-frame raycast cost
- Smooth transition: lerp `cutoffFrequency` over 0.2s to avoid hard filter pop

**Skip occclusion for:**
- 2D sounds (UI, music) — they have no spatial position
- Short SFX (< 0.5s duration) — player won't notice
- Very quiet sources (< 0.2 volume)

## Why this works
Low Pass Filter simulates acoustic absorption — real walls absorb high frequencies first. Volume drop simulates energy loss through walls. Both together create a convincing "behind wall" effect without spatial audio DSP.

## Trade-offs
- Doesn't handle sound leaking under doors, resonance, or diffraction — acceptable for low-poly games
- One raycast per source per 0.1s: at 20 sources = 200 raycasts/sec, not free but manageable
- `obstructionLayerMask` must include building walls but exclude terrain, foliage (foliage doesn't block sound perceptually)

## Variants
- **Distance-only**: no raycast, just volume falloff with distance (even simpler, good for background SFX)
- **HRTF-based occlusion**: Unity Resonance Audio plugin — accurate but expensive and complex
