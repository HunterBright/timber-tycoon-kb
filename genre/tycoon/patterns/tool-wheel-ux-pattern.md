---
name: tool-wheel-ux-pattern
description: Radial tool selector — hold Q opens wheel, mouse selects sector, LPM confirms / release Q cancels. Game runs (not paused) during selection.
metadata:
  type: pattern
  project: timber-tycoon
  suggested-category: genre/tycoon/patterns
  tags: [unity, ui, tool-wheel, fps, controls]
  severity: medium
  date: 2026-05-17
  status: draft
  applies_to: [unity-projects]
---

# ToolWheel UX Pattern

## When to use
FPS/third-person games with multiple tool/weapon types where switching should feel fast and natural, without opening a full inventory screen. Inspired by GTA V / Borderlands weapon wheels.

## Steps

**Input flow:**
1. Player holds Q → `ToolWheel.Show()`, cursor unlocked, mouse look disabled
2. Mouse moves → sector highlighted (nearest to cursor angle)
3. Left Mouse Button → `ToolManager.Equip(sector.tool)`, wheel closes
4. Release Q (no LMB) → cancel, wheel closes, current tool unchanged

**Input handling:**
```csharp
void Update() {
    if (Input.GetKeyDown(KeyCode.Q)) { wheelUI.Show(); playerController.canMove = false; }
    if (Input.GetKeyUp(KeyCode.Q))   { wheelUI.Hide(); playerController.canMove = true; }
    if (wheelUI.IsVisible && Input.GetMouseButtonDown(0))
        ConfirmSelection();
}

void ConfirmSelection() {
    var tool = wheelUI.GetHoveredTool();
    if (tool != null && tool.isUnlocked) ToolManager.Equip(tool);
    wheelUI.Hide();
    playerController.canMove = true;
}
```

**Critical:** game NOT paused during wheel (`Time.timeScale = 1`). Tycoon games → real-time pressure. Customers don't pause for you.

**Locked tools:** grayed out in wheel with unlock hint tooltip ("Unlock: Shovel after first stump"). Players see what's coming, know progression exists.

**Tool unlock sequence:**
1. Axe — unlocked at Quest 2 (FindAxe)
2. Shovel — unlocked after digging first stump
3. Planting tool — unlocked after first sapling collected

## Why this works
Hold-Q + mouse is faster than menu navigation. No game pause = real-time engagement. Visual wheel shows all tools at once — player learns available tools from day one.

## Trade-offs
- Mouse look disabled during wheel: must restore exactly on close. Use a "mouselock count" pattern to avoid partial restoration bugs
- `canMove = false` during wheel: if player holds Q too long while moving, character stops. Add momentum carry or brief grace period
- Controller support: wheel sectors map to face buttons (A/B/X/Y on Xbox) instead of mouse sectors. Plan for this if console support is planned

## Variants
- **Number hotkeys (1/2/3):** instant swap without visual wheel. Faster, less visual. Support both methods.
- **Scroll wheel:** scroll through tools like browser tabs. Works for 2-3 tools, breaks with 6+.

See also: [[universal-camera-lock-canmove-flag]], [[tool-viewmodel-child-of-camera-pattern]]
