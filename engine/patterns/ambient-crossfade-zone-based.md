---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, audio, ambient, crossfade, zones]
date: 2026-05-17
status: draft
---

# Ambient Crossfade Zone-Based Pattern

## When to use
Open-world or semi-open games where the player moves between distinct areas (forest, settlement, indoor, road). Without ambient zones, all areas sound identically sterile or identical.

## Steps

**AmbientManager (singleton):**
```csharp
public class AmbientManager : MonoBehaviour {
    AudioSource sourceA, sourceB;
    bool useA = true;

    public void TransitionTo(AmbientZoneSO zone, float fadeDuration = 1f) {
        var fadeOut = useA ? sourceA : sourceB;
        var fadeIn  = useA ? sourceB : sourceA;
        fadeIn.clip = zone.clip;
        fadeIn.volume = 0f;
        fadeIn.Play();
        StartCoroutine(CrossFade(fadeOut, fadeIn, zone.targetVolume, fadeDuration));
        useA = !useA;
    }
}
```

**AmbientZoneSO:**
```csharp
[CreateAssetMenu(menuName = "Audio/Ambient Zone")]
public class AmbientZoneSO : ScriptableObject {
    public AudioClip clip;
    public float targetVolume = 0.6f;
    public AudioReverbZone reverbOverride; // optional
}
```

**Zone detection:** trigger collider on zone boundaries. `OnTriggerEnter` → `AmbientManager.TransitionTo(newZone)`.

**Never-silent rule:** old source fades to 0.05f minimum (not 0f). Player always hears something — avoids jarring silence pockets at zone edges.

**Zones in Timber Tycoon:**

| Zone | Clip | Volume |
|------|------|--------|
| Forest (default) | birds + wind in trees loop | 0.7 |
| Sawmill interior | machine hum + wood creak | 0.5 |
| Road | wind + distant vehicle | 0.4 |
| River | water flow + birds | 0.65 |
| City (planned) | crowd murmur + traffic | 0.5 |

**Day/Night modifier:** AmbientManager modulates volume × `DayNightMultiplier` from TimeManager — nighttime forest is quieter with different clip (crickets vs. birds).

## Why this works
Two alternating AudioSources allow smooth crossfade without gap. Trigger-based zone detection is cheap (single event per zone boundary, not per-frame). The crossfade duration hides the transition, so players don't consciously notice the change.

## Trade-offs
- Overlapping trigger colliders: if player moves fast through a thin zone, multiple transitions fire quickly. Add minimum stay time (0.5s) before triggering to debounce
- Loop points: ambient clips must loop seamlessly — use audio editor to set clean loop points, avoid cross-fade artifacts at loop boundary
- 2D audio only: ambient is `spatialBlend = 0` (not positional). Positional ambient (river sound near river) is a separate `AudioSource` on the river object

## Variants
- **Distance-based mixing**: no trigger zones, volume is a function of distance to zone centers (smoother, more complex)
- **Layered ambient**: base ambient (forest) + additive layer (rain during weather) using additional source

See also: [[audio-reverb-zone-per-environment]], [[audio-manager-mixer-architecture]]
