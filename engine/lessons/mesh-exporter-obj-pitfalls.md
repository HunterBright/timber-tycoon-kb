---
type: lesson
project: timber-tycoon
suggested-category: engine/lessons
tags: [unity, editor-script, obj-export, locale, face-winding, precision]
severity: high
time_lost: "~4 hours"
date: 2026-05-17
status: draft
applies_to: ["unity-projects"]
---

# MeshExporter OBJ Pitfalls (3 Critical Bugs)

## Problem
Custom OBJ exporter in Unity Editor (for sending meshes back to Blender) has three bugs that all need fixing. Bug 1 fails silently — Blender shows an empty mesh with no error, making it the hardest to diagnose.

## Root cause

**Bug 1 — Polish locale decimal comma:**
`$"v {x} {y} {z}"` with Polish system locale writes `v 3,14 2,71 0,00`. OBJ spec requires decimal dot. Blender silently ignores all vertices, no error. Lost an afternoon to this.

**Bug 2 — Face winding order:**
Unity uses left-handed coordinate system, OBJ uses right-handed. Triangles spawn with inverted normals (back-face culled), mesh appears invisible.

**Bug 3 — Precision choice:**
Default float formatting is either wasteful (15 significant digits) or imprecise (`F2` loses sub-millimeter detail).

## Solution

**Bug 1 fix:** Use InvariantCulture formatting:
```csharp
using System.Globalization;
string.Format(CultureInfo.InvariantCulture, "v {0:F4} {1:F4} {2:F4}", x, y, z)
```

**Bug 2 fix:** Swap b and c in triangle output — use `triangles[i+2]` as b and `triangles[i+1]` as c.

**Bug 3 fix:** Use F4 (4 decimal places) — sub-millimeter precision, manageable file size.

Full corrected exporter: `Assets/Project/Scripts/Editor/MeshExporter.cs`
Menu: `Tools > Export Selected Mesh as OBJ`
Always log vertex count > 0 before declaring success.

## What didn't work
`F2` precision (lost detail on curved surfaces). Default string interpolation without CultureInfo (broke on any non-English locale).

## Transferability
Bug 1 (locale comma) affects any string formatting in Unity that writes numbers to files or network — applies beyond OBJ export to any data serialization. Always use `CultureInfo.InvariantCulture` for numeric file writes. Bugs 2 and 3 apply to any custom mesh serializer targeting right-handed coordinate systems.

## Related
- [Procedural textures must be baked](procedural-textures-need-bake.md)
- [FBX export standard settings](fbx-export-standard-settings-blender-to-unity.md)
