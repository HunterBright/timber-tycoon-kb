---
name: footstep-raycast-surface-detection
description: Footstep SFX surface detection via downward raycast — layer hit maps to SFX pool (grass/dirt/wood/stone/water)
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: engine/patterns
  tags: [unity, audio, footstep, raycast, terrain, sfx]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# Footstep Raycast Surface Detection

## When to use
Any FPS/third-person game where the player traverses multiple surface types (grass, wood floor, stone path). Single footstep sound across all surfaces breaks immersion immediately.

## Steps

**Trigger footstep event** (two options):
- **Distance-based:** track player `Vector3.Distance` from last step position, trigger at threshold (e.g., 1.5m walk, 1.0m run)
- **Animation event:** add event keyframe at foot-strike frame in walk/run animation → call `FootstepSystem.PlayFootstep()`

**Raycast + surface lookup:**
```csharp
void PlayFootstep() {
    Ray ray = new Ray(transform.position + Vector3.up * 0.1f, Vector3.down);
    if (Physics.Raycast(ray, out RaycastHit hit, 0.5f, terrainLayerMask)) {
        int layer = hit.collider.gameObject.layer;
        if (sfxTable.TryGetValue(layer, out AudioClip[] clips))
            audioSource.PlayOneShot(clips[Random.Range(0, clips.Length)]);
    }
}
```

**Lookup table** (`FootstepSO` — `Dictionary<int, AudioClip[]>`):

| Layer | Surface | Clips |
|-------|---------|-------|
| Default / Trees | Grass | 4–6 variants |
| Road layer 6 | Dirt/Gravel | 4–6 variants |
| Wood (sawmill floor, bridge) | Wood | 4–6 variants |
| Stone (cliff path) | Stone | 4–6 variants |

**Water:** check `WaterZone` trigger (separate) rather than raycast layer. Play splash + wade audio blend.

**Volume scaling:** `clip.volume = Mathf.Lerp(0.5f, 1.0f, normalizedSpeed)` — sprint footsteps louder than walk.

**Surface tagging:** assign correct layers to all walkable surfaces at scene setup time.

## Why this works
Raycast fires downward from just above the player's feet — always hits the exact surface being walked on. Lookup table per layer = any surface on that layer gets the right sound. 4–6 clips per surface prevents repetition.

## Trade-offs
- Layer-based (not tag): layers are faster (no string compare), but limited to 32 layers total — plan carefully
- Water needs special handling: WaterZone collider trigger is more reliable than raycasting into a water mesh
- Caves/tunnels: if ceiling is close, raycast might hit ceiling instead of floor — offset ray start to `feet.position` not `center.position`

## Variants
- **Texture-based** (terrain Texture2D sampling): accurate but only works with Unity Terrain, not custom meshes
- **Shoe effect per zone:** ambient step sound (reverberation) via `AudioReverbZone` — combine with per-surface clips for layered feel (see [[audio-reverb-zone-per-environment]])

See also: [[audio-manager-mixer-architecture]]
