---
title: blender-mcp bridge failure modes + headless CLI fallback
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-06-10'
project: Kerf - Sawmill Tycoon
tags:
- blender
- blender-mcp
- headless
- cli
- fallback
- debugging
- bake
applies_to: []
source: ''
promoted: '2026-07-30'
---

# blender-mcp bridge failure modes + headless CLI fallback

## Failure modes seen (and how to tell them apart)
Probe the bridge with a raw TCP client (port 9876, send
`{"type":"execute_code","params":{"code":"print(1)"}}`) instead of guessing:

1. **"Incomplete JSON response received" from the MCP layer** - the addon's
   responses are NUL-terminated (`...}\x00`); a strict parser chokes. A raw
   client that strips `\x00` reads them fine. This alone is recoverable.
2. **`{"status":"error","message":"Client timed out"}` from the addon itself** -
   the addon's server thread queued the command but Blender's MAIN THREAD never
   executed it. Cause: GUI is blocked (modal dialog, popup, render). No remote
   fix exists - needs a human to clear the dialog or restart Blender.

## The fallback that always works: headless CLI
`blender.exe --background --python script.py` runs a completely separate
Blender instance - no GUI, no addon bridge, fully deterministic, output
captured. Works for the entire asset pipeline: import FBX → procedural
material → Cycles bake (CPU, DIFFUSE pass_filter={'COLOR'}) → transforms →
save .blend → Cycles preview renders → FBX export. One script, repeatable -
re-running after a tweak costs seconds. Use `bpy.ops.wm.read_homefile(use_empty=True)`
for a clean scene (not factory reset).

Caveats: EEVEE renders may fail headless (no GL context) - use Cycles;
`uv.smart_project` works headless when the object is active+selected;
multi-material bakes need the bake image node added AND set active in EVERY
material's node tree.

## Bonus lesson: classifying mesh islands of an AI-generated prop
When splitting a monolithic AI mesh (Tripo) into regions by connected
components: the component with the MOST VERTICES is often NOT the main body
(a crumpled detail cluster can out-vert the smooth shell). Classify by
bounding-span instead (body = island spanning most of the height), then size
and position for the rest. Print a per-island table (verts, mean height,
span) and eyeball it before assigning materials.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[blender-mcp-interactive-remodel-loop|Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)]] - wspolne: blender-mcp, blender
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] - wspolne: bake, blender
- [[blender-headless-python-generation|Blender Headless Python Script Generation]] - wspolne: headless, blender
- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] - wspolne: bake, blender
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] - wspolne: bake, blender
- [[20260730-1217-bone-side-names-vs-axis-sign|Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)]] - wspolne: debugging, blender
<!-- /POWIAZANE:auto -->
