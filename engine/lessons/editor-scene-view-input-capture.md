---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, editor, scene-view, input, handle-utility, road-tool]
severity: high
time_lost: "~45 min"
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# Editor Scene View Input Capture Pattern

## Problem
Custom editor tools that want to capture mouse clicks in Scene View select the underlying object instead of running tool logic. "I click to add a waypoint, Unity selects the terrain instead."

## Root cause
Default Unity behavior: LMB in Scene View selects whatever is under the cursor. Editor tools must explicitly claim control via `HandleUtility.AddDefaultControl` AND consume the event via `e.Use()` — both are required.

## Solution
Required pattern in `OnSceneGUI`:
```csharp
if (isDrawing) {
    int controlId = GUIUtility.GetControlID(FocusType.Passive);
    HandleUtility.AddDefaultControl(controlId);
    
    if (e.type == EventType.Layout)
        HandleUtility.AddDefaultControl(controlId);
    
    if (e.type == EventType.MouseDown && e.button == 0 && !e.alt && !e.shift) {
        // ... tool logic here ...
        e.Use(); // CRITICAL: consume event
        return;
    }
}
```

Without `e.Use()`: tool logic runs AND Unity selects → both wrong.
Without `AddDefaultControl` in Layout: Unity's picking system grabs the event first.
ESC / right-click to exit drawing mode also needs `e.Use()`.

## What didn't work
Only calling `AddDefaultControl` without `e.Use()` — object still gets selected. Only calling `e.Use()` without Layout handling — event arrives late.

## Transferability
The "obvious in hindsight" trap for any first-time Unity editor tool author. Required pattern for ANY editor tool that intercepts mouse clicks in Scene View: road tools, placement tools, spline editors, custom gizmo tools.

## Related
- [Runtime vs Editor script separation](runtime-vs-editor-script-separation.md)
- [Custom Editor pattern for generators](../patterns/custom-editor-pattern-for-generators.md)
