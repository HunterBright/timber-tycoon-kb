---
type: pattern
project: timber-tycoon
suggested-category: engine/patterns
tags: [unity, minigame, mouse-drag, plankmaker, tempo]
date: 2026-05-17
status: draft
---

# Sliding Head Bandsaw — Mouse Drag Tempo Minigame

## When to use
Processing machines that simulate "feed material at consistent pace." Specifically designed for PlankMaker (log → planks via bandsaw). Fourth distinct mechanic in TT's minigame catalog alongside zone-click, circular-cursor needle, and hold-needle.

## Steps
1. Visual: sliding bandsaw head moves along track as player drags mouse left→right
2. Input: `Input.GetAxis("Mouse X")` per frame → accumulate velocity over 0.5s window
3. Optimal tempo: 1.5–2.5 cm/s average drag speed → PERFECT result
4. Measurement: rolling average of mouse delta over last 0.5s
5. Output per cycle:
   - PERFECT: 3 planks + 0 bark (best quality)
   - GOOD: 2 planks + 1 bark
   - BAD: 1 plank + 1 bark
6. Multi-cycle: `WoodTypeData.cuttingCycles` determines cycles per log (1/2/3 per species tier)

Thresholds: deviation < 5% = PERFECT, 5–15% = GOOD, >15% = BAD.

## Why this works
Rhythm-based input adds a "feel" category to the minigame catalog that zone-click and needle mechanics don't have. Player develops muscle memory for the correct drag tempo.

## Trade-offs
Mouse-only (no gamepad native feel). Requires velocity sampling code (not just instantaneous input).

## Variants
Same mechanic applicable to: saw operation (back-and-forth), carving, grinding — any "feed material at tempo" mechanic.
