---
type: pattern
project: timber-tycoon
suggested-category: genre/tycoon/patterns
tags: [unity, lighting, day-night, minecraft-style, low-poly]
date: 2026-05-17
status: draft
---

# Minecraft-Style Lighting (Static Overhead + Decorative Sun)

## When to use
Low-poly or stylized games with day/night cycle where visual sun position is decorative, not physically accurate. Prevents [[rotating-directional-light-day-night]] anti-pattern.

## Steps

**Directional Light setup (NEVER rotate it):**
```csharp
// Set once in Editor — do NOT rotate in code
// Rotation: approximately (45, 0, 0) — overhead-ish, slight angle for shadows
// This rotation NEVER changes at runtime
```

**DayNightCycle.cs — drives only intensity + color (not rotation):**
```csharp
void Update() {
    float t = timeOfDay; // 0-1 over 24 game-hours

    // Intensity: 0.15 (night) → 0.5 (dawn) → 1.0 (midday) → 0.5 (dusk) → 0.15 (night)
    directionalLight.intensity = EvaluateIntensity(t);

    // Color: cool blue (night) → warm orange (dawn) → white (midday) → red-orange (dusk)
    directionalLight.color = EvaluateColor(t);

    // Ambient: separate color track
    RenderSettings.ambientLight = EvaluateAmbient(t);

    // Skybox uniform — sun/moon position decorative only
    skyboxMaterial.SetFloat("_TimeOfDay", t);
}
```

**Minimum intensity rule** (TT design): never drop below 0.20 at night. Game is a sawmill — nighttime work should be possible without headlamp.

**Visual sun:** handled by skybox shader (see [[procedural-skybox-sun-moon-trick]]). Player sees sun rise/set but Directional Light doesn't move. The visual is decoupled from the physics.

**References:** Minecraft, Astroneer, Terraria — all use static or semi-static overhead light for stylized day/night.

## Why this works
At-rest Directional Light = consistent shadow angles throughout day/night. Light intensity and color still convey time of day powerfully. Players perceive "it's daytime/nighttime" from the colors and sky — they don't notice the fixed shadow direction.

## Trade-offs
- Fixed shadow direction: shadows from trees/buildings point the same way morning and evening. In stylized games, players don't notice or don't care. In realistic games, this would be wrong.
- Minimum 0.20 intensity: nighttime isn't dark enough for horror atmosphere. For horror games, drop to 0.05 but add a flashlight system.
- No atmospheric scattering: dawn/dusk color ramp is approximate. Full atmospheric simulation (Rayleigh scattering, Mie scattering) = significant shader complexity, out of scope for low-poly.

## Variants
- **Two light setup:** main (overhead, static) + secondary fill light (very low intensity, color = sky's complement). Adds soft counter-shadow on dark sides of objects.
- **Rotating with clamp:** rotate light but clamp to 30°–90° elevation range (above horizon always). Compromise — gets some angle variation without going horizontal.

See also: [[four-phase-weighted-smoothstep-day-night]], [[procedural-skybox-sun-moon-trick]], [[rotating-directional-light-day-night]]
