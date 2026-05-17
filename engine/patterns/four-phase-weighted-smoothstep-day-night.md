---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, urp, shader, day-night, smoothstep, weighted]
date: 2026-05-17
status: draft
---

# 4-Phase Weighted Smoothstep Day/Night Transition

## When to use
Day/night cycle systems that need smooth, gradual transitions between time-of-day lighting states — no hard cuts between day/night/dawn/dusk.

## Steps
Time `t ∈ [0, 1]` = 24-hour cycle (cycleDurationMinutes = 2).

4 phases with overlapping smoothstep ranges:
- night: `t ∈ [0.88, 0.12]` (wraps midnight)
- dawn: `t ∈ [0.12, 0.35]`
- day: `t ∈ [0.25, 0.80]`
- dusk: `t ∈ [0.70, 0.90]`

Each factor:
```glsl
float dayFactor = smoothstep(0.25, 0.40, t) * smoothstep(0.80, 0.65, t);
// ... similar for dawn, dusk, night
```

Final color = weighted sum:
```glsl
color = noonColor * dayFactor + dawnColor * dawnFactor + duskColor * duskFactor + nightColor * nightFactor;
```

Constraint: night minimum 55% intensity — NEVER fully dark (players need to see).

Ambient color tracks time directly (not sun rotation — see [[rotating-directional-light-day-night]] anti-pattern).

## Why this works
Overlapping smoothstep ranges mean each transition phase contributes a weighted blend rather than switching abruptly. Dawn color gradually fades in during late night, day color fades out at sunset. No visible hard cuts.

## Trade-offs
Requires careful range tuning to avoid visual artifacts (too-early dawn, clipped dusk). One-time tuning, then stable forever.

## Variants
Same technique for: indoor lighting by hour (office morning/afternoon/evening), weather tinting (sunny/cloudy/storm blend), seasonal color shifts.
