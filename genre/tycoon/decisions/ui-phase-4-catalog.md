---
title: UI Phase 4 Catalog (12 Systems)
type: decision
status: draft
confidence: medium
verified: ''
date: '2026-05-17'
project: Kerf - Sawmill Tycoon
tags:
- game-design
- ui
- roadmapa
- scope
- phase-4
applies_to:
- unity-projects
source: ''
description: 'UI Phase 4 - 12 systems catalog for post-MVP UX completion: HUD, Notification Queue, Tooltips, Cursor, Confirmation Dialogs, Visual Feedback, Typography, Icons, Statistics, Settings, Accessibility, Credits.'
severity: medium
suggested-category: genre/tycoon/decisions
name: ui-phase-4-catalog
---

# UI Phase 4 Catalog (12 Systems)

## Decision

**Scope:** Complete all 12 UI systems in Phase 4 (post-MVP, pre-Early Access). Document here to prevent scope creep ("wait, we never built a settings menu!").

## The 12 systems

| # | System | Description | Status |
|---|--------|-------------|--------|
| 1 | HUD | Money + day/time display | ✅ Complete |
| 2 | Notification Queue | Info/warning/achievement toasts (max 5 stacked) | ✅ Complete |
| 3 | Tooltip System | Inline help on UI hover (2-line, 8 subclasses) | ✅ Complete |
| 4 | Cursor Management | Dynamic crosshair states (3 states, auto-hide) | ✅ Complete |
| 5 | Confirmation Dialogs | Tak/Nie modal for irreversible actions | ✅ Complete |
| 6 | Visual Feedback | Floating text (+$, -$) + WorldProgressBarUI on machines | ✅ Complete |
| 7 | Typography / Fonts | Nunito, scaling, localization-safe | ✅ Complete |
| 8 | Ikony UI | 30 low-poly 3D icons via IconRenderer pipeline | ✅ Complete |
| 9 | Statistics / Journal | 11 stats + journal (max 200 entries) | ✅ Complete |
| 10 | Ustawienia gracza | Audio/graphics/controls settings (6 tabs) | ✅ Complete |
| 11 | Accessibility | Font scale, high contrast, 11 keyboard shortcuts | ✅ Complete |
| 12 | Credits Screen | Scrollable, 15 languages | ✅ Complete |

## Why catalog this

"We need a settings menu before Early Access" - caught late = emergency sprint. Caught here = planned work. All 12 were known risks; this document prevented surprises.

**Estimated effort per system:** 2-3 weeks average (UI work is inherently slow - layout, hover states, localization, edge cases).

## Lesson for future projects

Before MVP, catalog ALL UI systems you'll need at launch. Score each: required for MVP / required for EA / nice-to-have. Phase 4 = "required for EA, not MVP." Ship MVP without them, complete Phase 4 post-MVP.

See also: [[typography-accessibility-stack]], [[quest-highlight-pattern]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[player-built-vs-purchased-dichotomy|Player-Built vs. Purchased Dichotomy]] - wspolne: scope, game-design
- [[vfx-wycofane-decision|VFX Wycofane - Sawdust, Kurz, Liście Cut from MVP]] - wspolne: scope, game-design
<!-- /POWIAZANE:auto -->
