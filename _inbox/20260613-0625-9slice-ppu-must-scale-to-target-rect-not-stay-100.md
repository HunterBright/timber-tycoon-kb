---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, ugui, ui, 9-slice, sprite, pixels-per-unit, import, image-sliced]
severity: medium
time_lost: "~1 iteration (caught from in-game screenshot)"
date: 2026-06-13
applies_to: [unity-ugui]
---

# A large 9-slice sprite at PixelsPerUnit=100 breaks because its fixed corners exceed the panel

## Problem
A wooden 9-slice panel sprite (native 2048×1536, border L/R 490, T/B 367 px) was imported
with the default Pixels Per Unit = 100 and assigned to a code-built window that renders at
700×620 px. In-game it rendered as a 2×2 grid of bark "mini-panels" with the wood face
crushed into a tiny center rectangle — the classic broken-9-slice look.

## Root cause
In a UGUI Sliced Image the fixed corner/edge regions are drawn at
`borderPx × (canvasReferencePPU / spritePPU)` UI pixels (default canvasReferencePPU = 100).
At spritePPU = 100 the borders map 1:1, so the left+right corners occupied 490+490 = 980 UI px
and top+bottom 367+367 = 734 UI px. Both **exceed** the 700×620 panel, leaving negative space
for the stretchable middle. When corners can't fit, the slice layout collapses and the result
reads as repeated/overflowing corner art, not a clean frame. PPU=100 is fine for small UI
sprites but wrong for a big sprite with big borders dropped onto a normal-sized window.

## Solution
Raise the sprite's Pixels Per Unit so the corner extent is a safe fraction of the smallest
panel that uses it. Rule used: corners < ~70% of the panel (leaves ≥30% for the middle).
- Solve per axis: `borderSum × (100/N) < 0.70 × panelPx`.
  Width: `980×(100/N) < 0.70×700` → N > 200. Height: `734×(100/N) < 0.70×620` → N > 169.
- Take the most restrictive (N > 200), round up. Chose **N = 300** for margin AND to restore the
  artwork's native ~52% wood-face proportion (at 300: H border 163 px → center 53%, V border
  122 px → center 61%). Set `spritePixelsToUnits: 300` in the .png.meta; border values left
  unchanged; Unity reimports from the meta.
No code change: the factory already set `Image.type = Sliced` with default `fillCenter = true`;
the type was never the problem.

## What didn't work / red herrings
- Suspecting `Image.type` wasn't Sliced. It was — the factory sets it deterministically. The
  symptom mimics a type problem but is purely a scale problem.
- Lowering/raising border px would change the art framing; the correct lever is PPU, which
  scales the *drawn* border without touching the authored slice guides.

## Transferability
Applies to any Unity UGUI project using imported 9-slice sprites, especially when art is
authored at high native resolution (2K/4K) with proportionally large borders but consumed by
normal-sized runtime panels. Heuristic: a sprite with borders summing to B px needs
`PPU ≥ B / (0.7 × smallestPanelPx) × 100`. Note PPU is a per-sprite global, so it must satisfy
the SMALLEST panel that shares the sprite (at N=300 here, any panel ≥~470 px wide is safe).

## Related
- [[dim-scrim-must-not-reuse-9slice-panel-factory]] — sibling fix from the same reskin: the
  full-screen dim must not reuse the panel factory once a transparent 9-slice sprite exists.
