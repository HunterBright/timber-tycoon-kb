> NOTE: tier/quality claims in this aggregate are superseded by genre/tycoon/decisions/tier-system-foundation.md (regenerate to refresh).

# KB Build Package — Timber Tycoon Knowledge Base

**Generated:** 2026-05-17 by claude.ai (Opus 4.7) for autonomous execution by Claude Code.
**Target:** `<kb-root>\` (junction at `<project-root>\kb\`)
**Scope:** 168 candidates → ~150 unique .md files after dedup mergers.

---

## 1. Mission

Generate Knowledge Base from this spec without further input from Hunter.

Each entry has enough context that you (Code) can render the full `.md` file using the templates in `templates/{type}.md` (already exist). Do NOT search past conversations for additional context unless brief is incomplete (flag it instead).

**Definition of done:**
1. All ~150 .md files generated in correct paths
2. `MOC.md` updated with links grouped by category
3. Git committed per batch with message `feat(kb): batch N — {category}`
4. Final `KB_BUILD_REPORT.md` written with summary (entries done, dedups merged, skipped/flagged)

---

## 2. Workflow Protocol

### Templates (in `<kb-root>\templates\`)
- `lesson.md` — for Type A (technical know-how, gotchas, lessons learned)
- `pattern.md` — for Type B and C (reusable code patterns, design patterns)
- `decision.md` — for Type C (design decisions with reasoning, alternatives)
- `anti-pattern.md` — for Type A and D (what NOT to do, with example + fix)

**Type→Template mapping (when ambiguous):**
- Type A → lesson.md (default) OR anti-pattern.md (if framed as "don't do X")
- Type B → pattern.md
- Type C → pattern.md (if pattern) OR decision.md (if design choice)
- Type D → lesson.md (default for workflow rules) OR pattern.md (architecture patterns)

Per-entry, the brief includes explicit `Template:` field. Use that.

### Batch Processing

Process in 8 batches of ~20-25 entries each. After each batch:

````
git add <kb-root>/
git commit -m "feat(kb): batch N — {category}"
````

Then `/clear: TAK — kontekst pełny po N entries, fresh start dla batch N+1`.

**If interrupted mid-batch:** Check `KB_PROGRESS.md` for last completed `#ID`, resume from `#ID+1`. Don't re-create existing files (check filesystem first).

### File Generation Process (per entry)

1. Read entry spec from this document
2. Open template: `templates/{Template}.md`
3. Fill template sections from entry fields:
   - `# {Title}` → from `Title`
   - `**Tags:**` → from `Tags`
   - `## Brief` → from `Brief`
   - `## Details` → from `Details`
   - `## Why it matters` → from `Why it matters`
   - `## Sources` → list from `Sources` (claude.ai chat UUIDs as bullets)
   - `## Related` → links to `#Related` IDs (resolve to paths)
4. Save to `Path` from entry
5. Append `- [{Title}]({Path})` to appropriate section in `MOC.md`
6. Mark `#ID` complete in `KB_PROGRESS.md`

### MOC.md Structure (update incrementally)

```markdown
# MOC — Map of Content

## Engine
### Lessons (Type A — technical know-how)
- ...

### Patterns (Type B/C — reusable patterns)
- ...

### Anti-Patterns (Type A/D — what NOT to do)
- ...

## Genre
### Tycoon
#### Patterns
- ...
#### Decisions
- ...

### Survival (Eskimo Simulator cross-references)
#### Patterns
- ...
#### Decisions
- ...

### Roguelike
- ...

### PvP Multiplayer (Shooter design)
- ...

### Cross-Genre Decisions
- ...

## Workflow
### Claude Code
- ...
### MCP Tools
- ...
### 3D Models
- ...
### Asset Pipeline
- ...
```

### Failure Recovery

- If template missing → flag in report, skip entry
- If path already exists → check if file is from this batch (resume) OR merge if user-provided content present
- If brief references unknown UUID → still generate, leave Sources blank with `[needs source verification]`
- If you genuinely can't render an entry → add to "Flagged" section in `KB_BUILD_REPORT.md`, continue to next

---

## 3. Output Tree

```
<kb-root>\
├── engine\
│   ├── lessons\           (~30 files)
│   ├── patterns\          (~45 files)
│   ├── anti-patterns\     (~6 files)
│   └── decisions\         (planned, may be empty)
├── genre\
│   ├── tycoon\
│   │   ├── patterns\      (~22 files)
│   │   └── decisions\     (~12 files)
│   ├── survival\
│   │   ├── patterns\      (~1 file)
│   │   └── decisions\     (~1 file)
│   ├── roguelike\         (placeholder, 0-1 files)
│   ├── pvp-multiplayer\   (~1 file for shooter design)
│   └── cross-genre\
│       └── decisions\     (~1 file)
├── workflow\
│   ├── claude-code\       (~12 files)
│   ├── mcp-tools\         (~2 files)
│   ├── 3d-models\         (~6 files)
│   └── asset-pipeline\    (~7 files)
└── MOC.md                 (updated)
└── KB_PROGRESS.md         (created by Code, tracks done/todo)
└── KB_BUILD_REPORT.md     (created by Code at end)
```

---

## 4. Progress Tracker Template

Code creates `KB_PROGRESS.md` at start with this content:

```markdown
# KB Build Progress

Started: {timestamp}

## Batches
- [ ] Batch 1 (#001-#025): engine/lessons part 1
- [ ] Batch 2 (#026-#050): engine/lessons part 2 + engine/patterns part 1
- [ ] Batch 3 (#051-#075): engine/patterns part 2
- [ ] Batch 4 (#076-#100): engine/patterns part 3 + anti-patterns
- [ ] Batch 5 (#101-#125): genre/tycoon/patterns
- [ ] Batch 6 (#126-#145): genre/tycoon/decisions + survival + cross-genre
- [ ] Batch 7 (#146-#160): workflow/claude-code + mcp-tools
- [ ] Batch 8 (#161-#168): workflow/3d-models + asset-pipeline

## Completed Entries
(append `#NNN ✓ {title}` per completion)

## Flagged Entries
(append `#NNN ⚠️ {title} — reason` per flag)

## Dedup Mergers Applied
(append per merger executed)
```

---

## 5. Dedup Decisions (PRE-RESOLVED — execute as specified)

These mergers are pre-decided. Do NOT create separate files for the "merged into" IDs; instead, include their content in the target entry as additional sections or merged bullets.

| Merge | Into | Action |
|---|---|---|
| #096 (R4 StorageRack family v1) | #119 (R5 v2 with maxDistinctStacks) | Single file `engine/patterns/storage-rack-family-system.md`, includes both R4 and R5 details |
| sweep2 "CapsuleCollider script override" | #112 (specific trunk physics case) | Single file `engine/anti-patterns/script-overrides-prefab-collider.md`, use #112 as primary |
| #11 (MeshExporter Polish locale) | + #106 (face winding) + #107 (F4 precision) | Single file `engine/lessons/mesh-exporter-obj-pitfalls.md` with 3 subsections |
| sweep2 "TreeTypeData runtime injection" | #108 (general SO injection pattern) | Single file `engine/patterns/scriptableobject-runtime-injection.md`, use #108 as primary, reference #109 propagation |
| #93 + #94 + #95 (Typography stack) | Single file | `engine/patterns/typography-accessibility-stack.md` with 3 subsections (TextMeshPro rule / style guide / dynamic sizing) |

After execution: ~150 unique files from 168 candidates.

---

## 6. Entry Format Reference

Every entry below follows this exact shape:

```
### #NNN [★ if must-have] Title
- **Slug:** kebab-case-filename
- **Path:** category/subcategory/slug.md
- **Type:** A | B | C | D
- **Template:** lesson | pattern | decision | anti-pattern
- **Sources:** UUID1[, UUID2]
- **Related:** #X, #Y (cross-refs to other entry IDs, resolved at gen time)
- **Tags:** tag1, tag2, tag3
- **Dedup:** (optional — merger spec)

**Brief:** 1-2 sentences.

**Details:**
- Bullet, may include code in `inline`
- Bullet
- (numbers, parameters, file paths, error messages — verbatim where relevant)

**Why it matters:** 1 sentence anchored to TT context.
```

When rendering, treat the brief sections as authoritative content to expand into template body. Do not add fluff. Do not soften. Hunter's preferences apply: direct, terse, no hedging.

---

## 7. All Entries

The full catalog of 168 candidates follows. Process top-to-bottom in batches.

---

### BATCH 1 — engine/lessons (entries #001-#022)

#### #001 ★ Procedural textures don't export from FBX
- **Slug:** procedural-textures-need-bake
- **Path:** engine/lessons/procedural-textures-need-bake.md
- **Type:** A
- **Template:** lesson
- **Sources:** e55dbfd6-bb50, fda34b50-7e59
- **Related:** #003, #018
- **Tags:** blender, fbx, materials, baking, unity, urp

**Brief:** Blender procedural shaders (Wave/Voronoi/Noise + ColorRamp → Principled BSDF) don't carry through FBX export. Unity gets pink/gray materials.

**Details:**
- FBX format carries texture FILES, not shader graphs
- Procedural shaders exist only in Blender render context (Cycles/Eevee)
- Solution: bake to PNG before export — Albedo + Normal + AO maps
- Cycles GPU on RTX 4090: ~2-5min per asset
- Naming: `Mat_{ObjectName}_Bake.png` in `Assets/Models/{Category}/Textures/`
- Unity: URP Lit shader, point Base Map / Normal / Occlusion to baked PNGs
- File-size sanity: 512×512 for small props, 1024×1024 for buildings/vehicles

**Why it matters:** Skipping this step is the #1 reason for "where did my materials go" debugging sessions when bringing Blender assets into TT.

#### #002 ★ bake_space_transform + linked duplicates = 90° rotation injection
- **Slug:** bake-space-transform-linked-duplicates-rotation-bug
- **Path:** engine/lessons/bake-space-transform-linked-duplicates-rotation-bug.md
- **Type:** A
- **Template:** lesson
- **Sources:** (sweep2 — plank rack hierarchy)
- **Related:** #001, #005, #006
- **Tags:** blender, fbx, export, rotation, linked-duplicates

**Brief:** `bake_space_transform=True` in FBX export combined with linked duplicates injects 90° rotation into instances. Result in Unity: instances visually rotated 90° from original.

**Details:**
- Linked duplicates share mesh data but have independent transforms
- `bake_space_transform=True` bakes transform into mesh data on export
- The two interact: one duplicate's transform overrides others' visuals
- Workaround: use `PLAIN_AXES` empties as slot markers (e.g., `Fill_N_Slot_M`), not mesh duplicates
- For plank rack hierarchy: empties define slot positions, prefab instantiated at runtime via `PrefabUtility.InstantiatePrefab`
- Coord remap when transferring from Blender to Unity: `(X,Y,Z) → (X,Z,-Y)` + rotation reset

**Why it matters:** Cost us a session of debugging the plank rack. Hardcoded knowledge — never reproducible from documentation.

#### #003 ★ Pivot to single-material atlas for static props
- **Slug:** single-material-atlas-for-static-props
- **Path:** engine/patterns/single-material-atlas-for-static-props.md
- **Type:** C
- **Template:** pattern
- **Sources:** (sweep2 — multi-material FBX chaos)
- **Related:** #001, #004
- **Tags:** blender, materials, atlas, static-props, unity, slot-order

**NOTE:** Despite path being `patterns/`, this is in batch 1 because it's tightly coupled to lessons #001 and #004. Move at file generation time.

**Brief:** Multi-material FBX has unpredictable slot order in Unity. For static props (PelletBag, FertilizerBag, FirewoodBundle, PelletBox), use manual atlas + explicit UV layout in single material slot.

**Details:**
- Multi-material FBX: Unity reorders material slots non-deterministically across reimports
- Result: random materials assigned to wrong surfaces after reimport
- Solution for STATIC props: numpy-based atlas creation in Blender Python (load source PNG, resize regions, fill solid color blocks)
- Explicit per-face UV per region
- Single material slot from start — zero ambiguity
- Reserve procedural materials only for dynamic state (e.g., machine on/off, fill levels)
- Static props: PelletBag, PelletBox, FertilizerBag, FirewoodBundle

**Why it matters:** Single-material atlas is the difference between "ships in demo" and "ships in patch 3 after community complaints about random colors."

#### #005 FBX export standard settings (Blender → Unity)
- **Slug:** fbx-export-standard-settings-blender-to-unity
- **Path:** engine/lessons/fbx-export-standard-settings-blender-to-unity.md
- **Type:** A
- **Template:** lesson
- **Sources:** e55dbfd6-bb50, a45f1b8d-9d0b
- **Related:** #001, #002, #017
- **Tags:** blender, fbx, export, unity, scale-factor

**Brief:** Standard FBX export settings for Blender → Unity that prevent scale, rotation, and unit-conversion bugs.

**Details:**
- `axis_forward='-Z'`, `axis_up='Y'` (Unity convention)
- `apply_unit_scale=True`
- `apply_scale_options='FBX_SCALE_ALL'`
- `global_scale=1.0`
- `mesh_smooth_type='OFF'` (flat shading preserved)
- `use_mesh_modifiers=True`
- `bake_space_transform=True` (BUT see #002 for linked-duplicates caveat)
- Selected Objects: enabled
- Apply Transform: ENABLED before export (Ctrl+A → All Transforms)
- Unity Import Settings: Scale Factor=1, **Convert Units OFF** (critical), Bake Axis Conversion ON
- Validation: if Unity shows Scale 100 → export settings wrong; if invisible → normals flipped (recalculate outside)

**Why it matters:** These 7 lines are the difference between assets that "just work" and assets that require 15 minutes of debugging per import.

#### #006 Origin at bottom-center asset convention
- **Slug:** asset-origin-bottom-center-convention
- **Path:** engine/patterns/asset-origin-bottom-center-convention.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #005, #022
- **Tags:** blender, origin, pivot, unity, asset-convention

**Brief:** All TT assets follow bottom-center origin convention: trees = base of trunk, buildings = ground level center, vehicles = center of wheelbase, props = bottom center.

**Details:**
- Why: enables ground-snap via raycast without offset calculation
- Why: rotations look natural (asset rotates around base, not center)
- Why: stacking works (place asset on surface = pivot touches surface)
- Exception: PelletBag/FertilizerBag use exact bottom center (label-bottom)
- Tree adult: pivot at base of trunk, NOT center of mesh
- Vehicle: pivot at center of wheelbase on ground plane (y=0)
- Set in Blender: Object → Set Origin → Origin to 3D Cursor (cursor at desired position) or Origin to Geometry

**Why it matters:** Inconsistent pivots = floating assets, broken snapping, and Awful Bug Hunts™ when one tree behaves differently from another.

#### #011 ★ MeshExporter OBJ pitfalls (3 critical bugs)
- **Slug:** mesh-exporter-obj-pitfalls
- **Path:** engine/lessons/mesh-exporter-obj-pitfalls.md
- **Type:** A
- **Template:** lesson
- **Sources:** fd9cc5b6-bb50
- **Related:** #001, #017
- **Tags:** unity, editor-script, obj-export, locale, face-winding, precision
- **Dedup:** Merge #106 (face winding) + #107 (F4 precision) as subsections

**Brief:** Custom OBJ exporter in Unity Editor for sending meshes back to Blender has three critical bugs that all needed fixing.

**Details:**
- **Bug 1: Polish locale → decimal comma instead of dot.** `$"v {x} {y} {z}"` with Polish system locale writes `v 3,14 2,71 0,00`. Blender silently ignores all vertices, no error. Fix: `string.Format(CultureInfo.InvariantCulture, "v {0:F4} {1:F4} {2:F4}", ...)`. Add `using System.Globalization;`.
- **Bug 2: Face winding order.** Unity uses left-handed coordinate system, OBJ uses right-handed. Triangles spawn invisible (back-face culled). Fix: swap b and c — use `triangles[i+2]` as b and `triangles[i+1]` as c.
- **Bug 3: Precision choice.** Default float formatting is wasteful (15 digits) or imprecise (`F2` loses detail). F4 (4 decimal places) is the sweet spot — file size manageable, sub-millimeter precision preserved.
- Full corrected exporter saves to `Assets/Project/Scripts/Editor/MeshExporter.cs`
- Menu: `Tools > Export Selected Mesh as OBJ`
- Console log verifies vertex count > 0 before declaring success

**Why it matters:** Bug 1 fails SILENTLY — Blender shows empty mesh, no error, you blame your model. Lost an afternoon to this.

#### #012 ★ URP shadow cascade tuning for low-poly terrain
- **Slug:** urp-shadow-cascade-tuning
- **Path:** engine/lessons/urp-shadow-cascade-tuning.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #056, #057
- **Tags:** unity, urp, shadows, performance, road-artifacts, lighting

**Brief:** Default URP shadow cascade settings produce visible artifacts on TT roads (sharp transition lines, popping at distance). Specific tuning eliminates the artifact.

**Details:**
- Default URP Asset cascades: Count=4, Max Distance=50
- Symptom: visible sharp lines across roads where cascade boundaries fall
- Fix: Cascade Count 4 → **2**
- Fix: Last Cascade Border 4 → **10-15**
- Fix: Max Distance 50 → **80-100**
- Increase Depth Bias and Normal Bias on Directional Light to compensate for fewer cascades
- Trade-off: slight reduction in shadow detail for close-up objects, massive improvement for road quality

**Why it matters:** TT plays in third-person/first-person on flat-ish terrain — cascade artifacts are highly visible and break immersion. Fix is 30 seconds, ships forever.

#### #013 ★ NEVER save_scene or DestroyImmediate in Play Mode
- **Slug:** never-destructive-ops-in-play-mode
- **Path:** engine/lessons/never-destructive-ops-in-play-mode.md
- **Type:** D
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #014, #016
- **Tags:** unity, play-mode, scene-safety, data-loss, claude-code

**Brief:** Hard rule for all agents (Coplay MCP, editor scripts): NEVER `save_scene` or `DestroyImmediate` while Unity is in Play Mode. Result is permanent data loss.

**Details:**
- Unity Play Mode is sandboxed: changes don't persist unless explicitly saved
- Calling `save_scene` while in Play Mode saves the **Play Mode state** as the **edit-mode scene** — overwriting actual scene with runtime garbage
- `DestroyImmediate` in Play Mode destroys both the runtime instance AND the prefab reference if called on a prefab — permanent
- Coplay MCP agents must check Unity state before destructive operations
- Backup scene before any structural modification: `cp Assets/Demo_Scene.unity Assets/_Backup_{date}/Demo_Scene.unity`
- Recovery: pray to your version control (this is why we git commit per session)

**Why it matters:** This rule is written in blood. Two recoveries via git, one unrecoverable (rebuilt scene by hand). Never again.

#### #017 ★ Forward axis = -transform.right (Blender FBX quirk)
- **Slug:** forward-axis-blender-fbx-quirk
- **Path:** engine/lessons/forward-axis-blender-fbx-quirk.md
- **Type:** A
- **Template:** lesson
- **Sources:** 6967a837-dc86, 907b0de5-3c4c
- **Related:** #005, #018, #115
- **Tags:** unity, blender, fbx, transform, vehicle, forward-axis

**Brief:** Vehicles modeled in Blender and exported via FBX have local forward axis = `-transform.right` in Unity, not `transform.forward`. Affects movement, steering, parking alignment.

**Details:**
- Default expectation: `transform.forward` = mesh-front
- Reality after FBX from Blender: mesh-front = `-transform.right` (negative X-axis)
- Affects: `VehicleController` (AddForce direction), `NPCVehicle` (movement), `NPCParkingPDController` (slot forward alignment)
- Code convention: define `public Vector3 ForwardAxis => -transform.right;` and use everywhere
- Alternative: rotate model 90° in Blender before export (rejected — breaks existing prefabs)
- Detection: car drives sideways or refuses to move = check this first

**Why it matters:** This single axis quirk burned multiple debugging sessions. Now hardcoded in code conventions doc — never debug it again.

#### #018 ★ Self-collision compound BoxColliders → Physics.IgnoreCollision
- **Slug:** self-collision-compound-colliders-ignore
- **Path:** engine/lessons/self-collision-compound-colliders-ignore.md
- **Type:** A
- **Template:** lesson
- **Sources:** 6d0820c1-8138, 907b0de5-3c4c
- **Related:** #017, #019
- **Tags:** unity, physics, colliders, compound, rigidbody, vehicle

**Brief:** Vehicles with compound BoxColliders (Cabin + Flatbed + body) trigger self-collisions that explode the rigidbody on spawn. Fix: `Physics.IgnoreCollision` between all pairs at Awake.

**Details:**
- Compound colliders on same Rigidbody are NOT auto-excluded from collision detection
- Symptom: vehicle launches into air on spawn, or jitters in place
- Fix: at Awake, iterate all `Collider`s under root, call `Physics.IgnoreCollision(a, b, true)` for every pair
- Code: `var cols = GetComponentsInChildren<Collider>(); for (i...) for (j=i+1...) Physics.IgnoreCollision(cols[i], cols[j], true);`
- Caveat: only applies to colliders on the SAME rigidbody — child rigidbodies need separate handling
- Cabin/Flatbed BoxColliders should be triggers (see #019) — but even triggers can interact in compound setups

**Why it matters:** "Why does my car explode on spawn" — 40 minutes of debugging before finding this. Standard pattern now.

#### #023 Minimum turnFactor 0.3 for low-speed arcade steering
- **Slug:** minimum-turn-factor-arcade-steering
- **Path:** engine/lessons/minimum-turn-factor-arcade-steering.md
- **Type:** A
- **Template:** lesson
- **Sources:** 907b0de5-3c4c
- **Related:** #017, #022
- **Tags:** unity, vehicle, physics, arcade, steering, npc

**Brief:** Speed-dependent steering breaks at zero speed (turn factor → 0, can't steer to escape stuck state). Floor at 0.3.

**Details:**
- Common pattern: `turnFactor = Mathf.Clamp01(speed / maxSpeed)` for arcade feel (no power steering at standstill)
- Edge case: vehicle stuck against wall/curb at speed=0, can't rotate to escape because turnFactor=0
- Fix: `turnFactor = Mathf.Max(0.3f, Mathf.Clamp01(speed / maxSpeed))`
- Applies to both player vehicle (VehicleController) and NPC vehicles
- 0.3 = enough to rotate in place slowly, not enough to feel like power steering at full speed

**Why it matters:** Without the floor, NPCs get stuck in parking lots and player feels like the car is "dead" at low speed. Half-second fix, ships forever.

#### #058 ★ Vertex color gamma correction Blender→Unity
- **Slug:** vertex-color-gamma-correction-blender-to-unity
- **Path:** engine/lessons/vertex-color-gamma-correction-blender-to-unity.md
- **Type:** A
- **Template:** lesson
- **Sources:** fd9cc5b6-bb50, ff2af6d3-9b71
- **Related:** #059, #001
- **Tags:** blender, unity, vertex-colors, gamma, srgb, linear

**Brief:** Blender stores vertex colors in linear space; Unity reads them in sRGB space. Colors appear DARKER in Unity than in Blender — compensate by applying gamma correction at export.

**Details:**
- Blender vertex colors: linear RGB (0-1 floats, no gamma)
- Unity shaders (URP Lit, Custom/VertexColorLit): expect sRGB
- Without compensation: bright grass green in Blender → muddy olive in Unity
- Fix in Blender Python export script:
  ```python
  def linear_to_srgb(c):
      if c <= 0.0031308: return c * 12.92
      return 1.055 * (c ** (1/2.4)) - 0.055
  ```
- Apply per-channel before writing to vertex color attribute
- Or simpler approximation: `c ** (1/2.2)` (close enough for low-poly)
- Verify in Unity Play Mode (Game view, not Scene view) — Editor preview can lie

**Why it matters:** Without this, every terrain regen requires 5 iterations of "darker, no darker, no lighter" until you discover the gamma issue. Bake the conversion into the pipeline once.

#### #059 Stonowane kolory lesson — low-poly ≠ saturated
- **Slug:** desaturated-colors-for-low-poly
- **Path:** engine/lessons/desaturated-colors-for-low-poly.md
- **Type:** C
- **Template:** lesson
- **Sources:** fd9cc5b6-bb50
- **Related:** #058
- **Tags:** color, low-poly, aesthetic, terrain, immersion

**Brief:** First instinct for low-poly is to use saturated, bright colors (think mobile game). For TT (and most "cozy" low-poly games), reality is the opposite — desaturated, slightly muted palette.

**Details:**
- Saturated low-poly = "mobile shovelware" connotation; desaturated = "premium indie" (Slime Rancher, Townscaper)
- Grass: avoid #5B9947 (too bright); prefer #3D6B35 to #4A7841
- Rock: avoid #8C806B (too warm); prefer #5A524A (cooler, muted)
- Sand: avoid #D1C49E (jaundiced yellow); prefer #8B7D6B (natural beige)
- Color Variation parameter (vertex color jitter): keep LOW (~0.03), not high (0.06+). High variation = "pstrokaty" / patchy effect
- Test in-context (Play Mode at player eye height) NOT just Scene View flyover

**Why it matters:** Players don't articulate "your colors are too saturated" — they say "feels cheap" or "doesn't look polished." Fix at the palette level.

#### #033 ★ Read actual code before hypothesizing
- **Slug:** read-actual-code-before-hypothesizing
- **Path:** workflow/claude-code/read-actual-code-before-hypothesizing.md
- **Type:** D
- **Template:** lesson
- **Sources:** bccf249b-ec6b, 6967a837
- **Related:** #028, #034
- **Tags:** claude-code, debugging, workflow, anti-pattern-of-thought

**Brief:** When debugging, Claude (in chat) tends to hypothesize causes from symptoms. For TT, this is wrong default — first action is always: read the actual code involved.

**Details:**
- Symptom-pattern matching produces plausible but wrong hypotheses (e.g., "probably the collider is wrong" when actually the script overrides Inspector values — see #112)
- Correct workflow: ask for the file, read it, THEN diagnose
- Especially critical when Hunter has shown screenshots — visual ≠ ground truth, code is
- Saves entire debugging sessions where the wrong fix gets applied
- Doesn't replace #028 (context degradation) or #034 (3-level analysis), complements them

**Why it matters:** Recurring failure mode in long sessions. Codified as rule because pattern matching is seductive but unreliable.

#### #051 ★ LowPolyWater anti-pattern — sin(X) + sin(Z) = side waves
- **Slug:** low-poly-water-side-wave-antipattern
- **Path:** engine/anti-patterns/low-poly-water-side-wave.md
- **Type:** A
- **Template:** anti-pattern
- **Sources:** 6d0820c1-8138, fd9cc5b6-bb50
- **Related:** #052, #053
- **Tags:** unity, urp, shader, water, vertex-displacement

**Brief:** First-pass water shader applies sinusoidal vertex displacement equally on X and Z axes. Result: water "wobbles" perpendicular to flow, looks like trapped pond not flowing river.

**Details:**
- Symptom: river surface oscillates sideways toward banks, breaks "river current" illusion
- Root cause: `displacement = sin(time + X*freq) + sin(time + Z*freq)` — symmetric in X/Z
- Fix: anisotropic displacement aligned to flow direction
  - Main wave along flow axis (X-axis if flow is +X)
  - Side wave damping factor `_SideWaveDamping = 0.1` (10% of main amplitude max)
  - Add noise scroll for organic variation (`_FlowNoiseScale = 3.0`)
- Properties to expose: `_WaveSpeed`, `_WaveHeight`, `_SideWaveDamping`, `_FlowDirection (Vector)`
- Full corrected shader: `Assets/Project/Shaders/LowPolyWater.shader`

**Why it matters:** Water is one of the first things players notice. Wrong = "game looks broken." Right = "wow, this is cozy."

#### #078 ★ Separate-objects mapping rule — heightmap is not for cliffs
- **Slug:** separate-objects-mapping-rule
- **Path:** engine/lessons/separate-objects-mapping-rule.md
- **Type:** A
- **Template:** lesson
- **Sources:** 6967a837-dc86
- **Related:** #077, #079
- **Tags:** unity, terrain, heightmap, cliffs, caves, level-design

**Brief:** Terrain heightmap cannot represent overhangs, caves, vertical cliffs, or under-passages. Build these as separate FBX objects placed on the heightmap.

**Details:**
- Heightmap = 2D function Y = f(X, Z). Single Y per (X,Z).
- Cannot represent: cave (two Ys at same XZ), overhang (cliff sticking out), tunnel
- TT's map: heightmap = LowPolyTerrain.fbx (one Y per cell). Cliff + Cave + Mountains + Bridge + Waterfall = separate FBX objects
- Each object owns its mesh, materials, colliders
- Place once, lock position ("DO NOT MOVE" convention — see #080)
- Setup scripts orchestrate placement: `RebuildFullMap.cs` calls `SetupCliff.cs`, `SetupBridge.cs`, etc.

**Why it matters:** Trying to do cliffs in heightmap destroys geometry (degenerate triangles, sub-zero Y values). Separate objects = clean meshes, proper colliders, designer control.

#### #101 ★ Road.cs runtime vs RoadTool.cs editor folder separation
- **Slug:** runtime-vs-editor-script-separation
- **Path:** engine/lessons/runtime-vs-editor-script-separation.md
- **Type:** A
- **Template:** lesson
- **Sources:** fd9cc5b6-bb50
- **Related:** #105, #060
- **Tags:** unity, editor-script, runtime, assemblies, road-tool

**Brief:** Runtime components (`Road.cs` with `[MonoBehaviour]`) MUST live in `Assets/Project/Scripts/`. Editor extensions (`RoadTool.cs` with `[CustomEditor]`) MUST live in `Assets/Project/Scripts/Editor/`. Mixing breaks builds.

**Details:**
- Unity creates an `Editor` assembly from any folder named `Editor`, separate from runtime
- `using UnityEditor;` is only valid in Editor assembly (or wrapped in `#if UNITY_EDITOR`)
- Runtime script in Editor folder: never gets attached to GameObjects at build time → silent failure
- Editor script in runtime folder: build error "UnityEditor namespace not found"
- Folder structure for Road system:
  - `Assets/Project/Scripts/Road.cs` (runtime, has waypoints, generates mesh)
  - `Assets/Project/Scripts/Editor/RoadTool.cs` (CustomEditor, Scene View handles, draw waypoints)
- Create `Editor/` folder if it doesn't exist

**Why it matters:** "Build works in editor but breaks in standalone" — this is one of the causes. Bake the convention into the project structure.

#### #105 ★ Editor Scene View input capture pattern
- **Slug:** editor-scene-view-input-capture
- **Path:** engine/lessons/editor-scene-view-input-capture.md
- **Type:** A
- **Template:** lesson
- **Sources:** fd9cc5b6-bb50
- **Related:** #101, #060
- **Tags:** unity, editor, scene-view, input, handle-utility, road-tool

**Brief:** Custom editor tools that want to capture mouse clicks in Scene View must use `HandleUtility.AddDefaultControl` + `e.Use()`, otherwise Unity selects objects instead of running tool logic.

**Details:**
- Default behavior: LMB in Scene View selects object under cursor
- Bug symptom: "I click to add waypoint, Unity selects the terrain instead"
- Required pattern in `OnSceneGUI`:
  ```csharp
  if (isDrawing) {
      int controlId = GUIUtility.GetControlID(FocusType.Passive);
      HandleUtility.AddDefaultControl(controlId);
      
      if (e.type == EventType.Layout)
          HandleUtility.AddDefaultControl(controlId);
      
      if (e.type == EventType.MouseDown && e.button == 0 && !e.alt && !e.shift) {
          // ... do tool logic ...
          e.Use(); // CRITICAL: consume event
          return;
      }
  }
  ```
- Without `e.Use()`: tool logic runs AND Unity selects → both wrong
- Without `AddDefaultControl` in Layout: Unity's picking system grabs event first
- ESC or right-click to exit drawing mode (also needs `e.Use()`)

**Why it matters:** This is the "obvious in hindsight" trap for first-time editor tool authors. Burned 45 minutes on RoadTool. Standard pattern now.

#### #113 CapsuleCollider direction axis cheatsheet
- **Slug:** capsule-collider-direction-axis-cheatsheet
- **Path:** engine/lessons/capsule-collider-direction-axis.md
- **Type:** A
- **Template:** lesson
- **Sources:** a45f1b8d-9d0b
- **Related:** #112, #018
- **Tags:** unity, physics, collider, capsule, axis

**Brief:** `CapsuleCollider.direction` parameter cheatsheet: 0 = X-axis (horizontal log/lying cylinder), 1 = Y-axis (default vertical), 2 = Z-axis.

**Details:**
- Direction 0: capsule oriented along local X-axis (good for: fallen trunk, ship hull, lying objects)
- Direction 1: capsule oriented along local Y-axis (default — character controllers, standing trees, vertical poles)
- Direction 2: capsule oriented along local Z-axis (good for: torpedoes, projectiles, forward-pointing objects)
- For fallen trunk after tree fall: `direction = 0`, radius = 0.15, height = 3.5 (matches model exported with length along X)
- For standing tree: `direction = 1`, radius = 0.5, height = 5
- For Blender models: check which Blender axis is the long axis of the mesh — that determines Unity direction value after FBX axis remap

**Why it matters:** Default direction (1) often wrong for lying objects → collider doesn't match mesh → physics breaks (see #112).

#### #127 ★ Cylindric beams vs rectangular beams visual contrast
- **Slug:** cylindric-beams-visual-contrast
- **Path:** engine/lessons/cylindric-beams-visual-contrast.md
- **Type:** C
- **Template:** lesson
- **Sources:** fda34b50-7e59
- **Related:** #129, #130
- **Tags:** blender, low-poly, modeling, architecture, visual-design

**Brief:** Rectangular beams against rectangular walls = visually merged, no separation. Cylindric beams (logs) against rectangular walls = clear structural read, fits low-poly wood-industrial aesthetic.

**Details:**
- Visual contrast principle: differentiate structural layers with different primitive shapes
- TT sawmill: walls are rectangular planes, pillars are cylinders. If beams were also rectangular, walls + beams blend together
- Fix: beams as cylinders (logs), 0.15×0.15 cross-section × N meters length, rotated to lie horizontal
- Naming: `Beam_North_Log`, `Beam_East_Top_Log` (suffix `_Log` confirms cylinder)
- Pillars also cylindric (same shape language) → beams + pillars feel connected; walls feel like infill
- Industrial wood aesthetic emerges from this hierarchy: support structure (cylinders) + cladding (planes)

**Why it matters:** First pass had rectangular beams, looked muddy and amateur. Switching to cylinders was 20 minutes of work, the building immediately read as "wooden sawmill" instead of "generic shed."

#### #131 TreeState + StumpState enums state machine
- **Slug:** tree-stump-state-machine-enums
- **Path:** engine/patterns/tree-stump-state-machine-enums.md
- **Type:** A
- **Template:** pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** #132, #108
- **Tags:** unity, state-machine, enum, tree-cutting, gameplay

**Brief:** Tree life-cycle managed by two state enums: `TreeState` (standing → felled → debranched → cut into logs) and `StumpState` (in-ground → digging → ejected → collected).

**Details:**
- `TreeState`: `Standing, Chopping, Fallen, Debranching, ReadyToCut, Cutting, Done`
- `StumpState`: `InGround, Digging, Ejected, Collected`
- Each state has clear transition triggers (player interaction, minigame complete, time elapsed)
- ChoppableTree.cs holds current state, exposes events for state changes
- GameObject children get enabled/disabled based on state (e.g., axe minigame UI only during `Chopping`)
- Save/load preserves state (ISaveable) — partial progress on a tree survives reload

**Why it matters:** Without explicit state, code becomes "if branchesRemoved && !logsSpawned" chains that become unmaintainable by entry 4. Enum makes state visible and queryable.

#### sweep2-1 ChoppableTree multi-type naming convention
- **Slug:** choppable-tree-multi-type-naming-convention
- **Path:** engine/patterns/choppable-tree-multi-type-naming-convention.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #108, #109, #111
- **Tags:** unity, prefabs, naming, scriptableobject, tree-types

**Brief:** Multi-species tree system uses strict naming convention so TreeTypeData SO can reference prefabs generically: `{TreeType}_{Suffix}` for all related assets.

**Details:**
- TreeTypeData SO has `treeType` string field (e.g., "Spruce", "Birch", "Maple", "Oak")
- All related prefabs: `{treeType}_Stump`, `{treeType}_Trunk_Fallen`, `{treeType}_Log_N`, `{treeType}_Sapling`, `{treeType}_Small`, `{treeType}_Medium`, `{treeType}_Adult`
- Material naming: `Mat_{treeType}_Trunk`, `Mat_{treeType}_Stump`, `Mat_{treeType}_Leaves`
- Allows TreeTypeData to inject prefab refs via Inspector without code changes per species
- Adding new species: create folder `Assets/Models/Trees/{NewType}/`, populate models with naming convention, create SO

**Why it matters:** Consistency in naming enables the "zero code changes per species" promise of #111. Without it, every new species requires manual prefab field updates everywhere.

#### sweep2-2 DirtClump radius/scale tuning — Inspector overwritten by code
- **Slug:** dirtclump-inspector-overwritten-by-code
- **Path:** engine/anti-patterns/script-overrides-prefab-inspector-values.md
- **Type:** D
- **Template:** anti-pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #112, #022
- **Tags:** unity, prefabs, inspector, scripts, debugging

**Brief:** Setting values in Inspector that get overwritten by code at runtime is a recurring source of confusion — "I set it to 0.5 but in Play Mode it's 0.05." Worst offender: DirtClump radius/scale.

**Details:**
- Pattern: prefab has Inspector field with value, code in Awake/Start overwrites with hardcoded value
- Example: DirtClump prefab has `radius = 0.5` in Inspector, `CreateClump()` sets `radius = 0.05` from code
- Designer experience: "I'm editing the value but nothing changes" → wasted iteration cycles
- Two fixes:
  1. **Remove the hardcoded override** — trust the Inspector value (preferred when value is designer-tunable)
  2. **Read from a SO or const** — eliminate ambiguity, designer knows the value is code-controlled (preferred when value is mathematical/derived)
- Detection: if Inspector edit produces no in-game change, grep for the field name + `=` in code
- Generalization (see also #112 CapsuleCollider override): NEVER overwrite Inspector values from code unless you mean to

**Why it matters:** Designer wastes 20 minutes on every encounter. Codifying this prevents the next instance.

#### sweep2-3 Race condition: Start() reads SO before parent sets it
- **Slug:** race-condition-start-vs-instantiate-parameter
- **Path:** engine/anti-patterns/race-condition-start-vs-instantiate-parameter.md
- **Type:** D
- **Template:** anti-pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #108, #109
- **Tags:** unity, lifecycle, race-condition, instantiate, scriptableobject

**Brief:** `Instantiate(prefab)` triggers `Start()` immediately. If parent calls `Start()`-reliant initialization AFTER Instantiate (e.g., `gt.treeTypeData = ...`), the child's Start() ran with null SO.

**Details:**
- Buggy pattern:
  ```csharp
  GameObject tree = Instantiate(prefab);
  GrowingTree gt = tree.GetComponent<GrowingTree>();
  gt.treeTypeData = data; // TOO LATE — Start() already ran with null
  ```
- Symptom: `NullReferenceException` in `GrowingTree.Start()` or default values used instead of injected SO
- Fix 1 (preferred): public initialization method, called after assignment
  ```csharp
  GameObject tree = Instantiate(prefab);
  GrowingTree gt = tree.GetComponent<GrowingTree>();
  gt.treeTypeData = data;
  gt.InitializeWithSO(); // explicit init after parent has set fields
  ```
- Fix 2: use Awake instead of Start for SO-independent setup, defer SO-reading to first Update or coroutine
- Fix 3: pass SO into instantiation via factory method: `TreeFactory.Spawn(prefab, data)`
- General rule: child lifecycle methods should NOT assume parent-injected fields are set

**Why it matters:** This bit us during TreeTypeData migration. Subtle because Inspector-set values mask the bug — only fails when injected at runtime.

#### sweep2-4 Tag assignment in code (CompareTag) vs Inspector
- **Slug:** tag-assignment-code-vs-inspector
- **Path:** engine/lessons/tag-assignment-code-vs-inspector.md
- **Type:** A
- **Template:** lesson
- **Sources:** e55dbfd6-bb50
- **Related:** sweep2-2, #022
- **Tags:** unity, tags, comparetag, prefabs, runtime-spawning

**Brief:** Tags assigned in Inspector get lost on runtime-spawned objects if prefab doesn't have the tag set. Default: assign tags in code at spawn time AND on prefab, or fail to detect via `CompareTag`.

**Details:**
- Pattern: spawn DirtClump from code, expect `other.CompareTag("DirtClump")` in player trigger to detect it
- Bug: prefab's tag is "Untagged" → spawned instances are also Untagged → CompareTag returns false
- Two fixes:
  1. Set tag on prefab in Inspector (Project view → prefab → Tag dropdown → DirtClump)
  2. Assign tag in spawning code: `clump.tag = "DirtClump"`
- Pitfall: tag must exist in Project Settings → Tags first; otherwise both methods fail silently
- For tags used by gameplay code, add a string constant or define them in a Tags.cs file: `public static class Tags { public const string DirtClump = "DirtClump"; }`

**Why it matters:** Easy to forget. Spend 10 minutes wondering why pickup isn't registering, find tag is Untagged.


---

### BATCH 2 — engine/patterns part 1 (entries #023-#047)

#### #019 Cabin/Flatbed BoxColliders as triggers
- **Slug:** vehicle-interaction-zones-as-triggers
- **Path:** engine/patterns/vehicle-interaction-zones-as-triggers.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6d0820c1-8138, 6967a837-dc86
- **Related:** #018, #116
- **Tags:** unity, vehicle, colliders, triggers, interaction

**Brief:** Vehicle parts that player interacts with (Cabin = enter, Flatbed = load/unload) are BoxColliders set as triggers, separate from the physics BoxCollider on root.

**Details:**
- Vehicle root has solid BoxCollider for physics (collision with ground, walls, NPCs)
- Cabin child: trigger BoxCollider with `VehicleInteractable` (E to enter)
- Flatbed child: trigger BoxCollider with `FlatbedInteractable` (E to load/unload logs)
- Both children on layer `Interactable` (layer 3) for player raycast
- Triggers ignore physics but fire OnTriggerEnter for interaction detection
- Auto-setup via Editor: `Tools > Setup All Vehicles` configures all this from FBX naming convention

**Why it matters:** Single solid collider with raycast hit-test all surfaces is fragile (cabin hit feels different than flatbed). Triggers + dedicated scripts = clean interaction zones.

#### #021 ★ Vehicle save bug — Awake-init for ISaveable with dependencies
- **Slug:** awake-init-for-isaveable-with-dependencies
- **Path:** engine/patterns/awake-init-for-isaveable-with-dependencies.md
- **Type:** B
- **Template:** pattern
- **Sources:** 6967a837-dc86
- **Related:** #068, sweep2-3
- **Tags:** unity, isaveable, lifecycle, awake, start, save-system

**Brief:** ISaveable components that depend on other components (ServiceLocator services, sibling references) must initialize in Awake, not Start. SaveManager runs LoadSaveData in Start phase — too late if your dependencies aren't ready.

**Details:**
- SaveManager Start: iterates all ISaveable, calls `LoadSaveData(json)`
- If your ISaveable.Start does field setup, save data loads BEFORE setup completes → fields overwrite with restored values, then Start clobbers them with defaults
- Fix: move dependency wiring to Awake (Services.Register, sibling cache)
- Save load in Start now has fully-initialized state to write into
- Specific case: VehicleStorage Awake-init for prefab refs + Services.Get<EconomyManager>()
- Order of Unity lifecycle: Awake (all instances) → OnEnable → Start (all instances) → first Update
- ISaveable.LoadSaveData triggered in SaveManager.Start (after all Awakes, but Start order undefined)

**Why it matters:** Vehicle would respawn with empty flatbed on every load — even though save file had 5 logs. Awake-init fix made saves actually work.

#### #022 ★ GetOrAdd component pattern
- **Slug:** get-or-add-component-pattern
- **Path:** engine/patterns/get-or-add-component-pattern.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6967a837-dc86, 907b0de5-3c4c
- **Related:** #019, #021
- **Tags:** unity, components, extension-method, refactoring

**Brief:** Defensive component-add pattern: extension method `GetOrAddComponent<T>()` returns existing or adds new. Prevents duplicate components from re-running setup scripts.

**Details:**
- Common bug: `gameObject.AddComponent<T>()` called twice → two instances, both run Awake/Start, second one wins, first one orphaned
- Specific TT case: NPCVehicle.AddComponent called on spawn → if spawn re-runs (parking respawn), duplicate component
- Solution as extension method:
  ```csharp
  public static T GetOrAddComponent<T>(this GameObject go) where T : Component {
      T comp = go.GetComponent<T>();
      return comp ?? go.AddComponent<T>();
  }
  ```
- Place in `Assets/Project/Scripts/Extensions/GameObjectExtensions.cs`
- Use everywhere you'd write `AddComponent`: `vehicle.GetOrAddComponent<NPCVehicle>()`

**Why it matters:** Eliminates "spawn duplicates component" entire category of bugs in one extension method. Standard pattern in TT codebase now.

#### #037 ★ Storage migration: Primary new + Legacy fallback with TODO Sprint X
- **Slug:** storage-migration-primary-plus-legacy-fallback
- **Path:** engine/patterns/storage-migration-primary-plus-legacy-fallback.md
- **Type:** C
- **Template:** pattern
- **Sources:** bccf249b-ec6b
- **Related:** #133, #038
- **Tags:** unity, refactoring, migration, save-compat, technical-debt

**Brief:** When migrating storage systems (WarehouseManager → StorageManager), don't delete legacy access immediately. Pattern: primary path uses new system, fallback to legacy if migration incomplete, with TODO Sprint X to remove fallback.

**Details:**
- Bug to avoid: hard-delete WarehouseManager refs → legacy save files break, partial-migration code paths fail
- Pattern in code:
  ```csharp
  // Sprint C2 — migration to StorageManager
  int removed = StorageManager.Instance.ConsumeOneLog(species);
  if (removed == 0) {
      // TODO Sprint C5: remove fallback once all systems migrated
      removed = WarehouseManager.Instance?.ConsumeOneLog(species) ?? 0;
  }
  ```
- Save compatibility: old save files migrated at load time (read legacy keys, write new keys, remove legacy)
- Migration sprint markers in code: `// TODO Sprint X:` makes the cleanup visible in grep
- Remove fallback only when grep shows zero hits on legacy class

**Why it matters:** Hard migrations break old saves. Soft migrations preserve player progress. Costs ~50 lines of fallback code, gains: refactor isn't a horror.

#### #038 Before-delete legacy class checklist
- **Slug:** before-delete-legacy-class-checklist
- **Path:** engine/patterns/before-delete-legacy-class-checklist.md
- **Type:** A
- **Template:** pattern
- **Sources:** bccf249b-ec6b
- **Related:** #037, #133
- **Tags:** refactoring, deletion, checklist, technical-debt

**Brief:** Checklist before deleting a legacy class: grep all `.cs` files + scene/prefab references + ScriptableObject references. Forget one → silent breakage.

**Details:**
- Step 1: grep class name in all .cs files: `grep -r "WarehouseManager" Assets/Project/Scripts/`
- Step 2: search YAML scene/prefab files (text-readable YAML): `grep -r "WarehouseManager" Assets/**/*.unity Assets/**/*.prefab`
- Step 3: ScriptableObject references — open `.asset` files in text editor, search for class name
- Step 4: AssemblyDefinition references (`.asmdef`) — class may be referenced by assembly name
- Step 5: ONLY then delete class file
- Recovery: if you deleted prematurely, scene files show `m_Script: {fileID: 11500000, guid: 0000000000000000…}` (missing script) — manually remove via Inspector (see #014)

**Why it matters:** "I deleted X and now my scene is broken" — usually a step skipped in this checklist. Five-minute discipline saves an hour of recovery.

#### #039 ScriptableObject runtime-assigned references
- **Slug:** scriptable-object-runtime-assigned-references
- **Path:** engine/patterns/scriptable-object-runtime-assigned-references.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #108, #109
- **Tags:** unity, scriptableobject, runtime, references

**Brief:** SOs configured at design time but referenced at runtime via code injection: pass SO reference to instances during spawning, not via Inspector binding.

**Details:**
- Design-time pattern: prefab has SO field set in Inspector — works for fixed configs (player stats SO)
- Runtime pattern: spawned object gets SO assigned by spawner (e.g., ChoppableTree spawned → DiggableStump assigned TreeTypeData)
- Why runtime: multiple species share same prefab, only difference is SO data → no prefab variants needed
- Code pattern:
  ```csharp
  GameObject stump = Instantiate(stumpPrefab);
  DiggableStump ds = stump.GetComponent<DiggableStump>();
  ds.treeTypeData = this.treeTypeData; // pass from parent
  ds.Initialize(); // see sweep2-3 for race condition fix
  ```
- Avoid: singleton lookup of SOs (creates global state)
- Prefer: explicit parameter passing through spawn chain

**Why it matters:** Enables data-driven content scaling (see #111 "zero code changes per species") without prefab explosion.

#### #040 ★ Diegetic 3D button raycast (root BoxCollider toggle)
- **Slug:** diegetic-3d-button-raycast-pattern
- **Path:** engine/patterns/diegetic-3d-button-raycast.md
- **Type:** C
- **Template:** pattern
- **Sources:** (from 42-originals — minigame/UI domain)
- **Related:** #041, #061
- **Tags:** unity, ui, raycast, minigame, interaction, diegetic

**Brief:** Diegetic 3D buttons (physical buttons in world, not UI canvas) detected by player camera raycast against root BoxCollider. Toggle collider enabled state to enable/disable button without removing it.

**Details:**
- 3D button = mesh in world (e.g., button on machine panel) with BoxCollider
- Player camera raycast on E press: `Physics.Raycast(camera.position, camera.forward, hit, range, interactableLayer)`
- Check hit.collider's parent for `IInteractable` component, call `OnInteract()`
- Toggle pattern: `boxCollider.enabled = state` instead of disabling the GameObject (preserves visibility, disables interaction)
- Convention: root BoxCollider sized to entire button surface (forgive imprecise aim)
- Same pattern for kiosk (see #135), countertop interactables, machine panels

**Why it matters:** Diegetic UI feels immersive (player presses a button vs. canvas overlay). This pattern makes them work without complex UI systems.

#### #041 Camera lock pattern (save world coords → lerp → restore)
- **Slug:** camera-lock-save-lerp-restore
- **Path:** engine/patterns/camera-lock-save-lerp-restore.md
- **Type:** C
- **Template:** pattern
- **Sources:** (from 42-originals — minigame/UI domain)
- **Related:** #040, #117
- **Tags:** unity, camera, lerp, minigame, restore, cinematic

**Brief:** During minigames or cutscenes, lock camera to a defined position by saving current world coords, lerping to target, and restoring on exit.

**Details:**
- Save state at minigame start:
  ```csharp
  _savedCamPos = playerCamera.transform.position;
  _savedCamRot = playerCamera.transform.rotation;
  _savedCamParent = playerCamera.transform.parent;
  ```
- Detach from player, lerp to target over ~0.3s:
  ```csharp
  while (t < 1f) {
      t += Time.deltaTime / 0.3f;
      playerCamera.transform.position = Vector3.Lerp(_savedCamPos, targetPos, t);
      playerCamera.transform.rotation = Quaternion.Slerp(_savedCamRot, targetRot, t);
      yield return null;
  }
  ```
- Restore on exit: reverse lerp, then re-parent to saved parent at saved position
- During lock: PlayerController.canMove = false (see #117)
- Use `Cinemachine` if available; this pattern for projects without it

**Why it matters:** Manual camera control = bugs (snapping, jitter, lost state). Lerp pattern provides smooth feel + bulletproof restore.

#### #044 Sliding head bandsaw — mouse drag tempo minigame
- **Slug:** sliding-head-bandsaw-mouse-drag-tempo-minigame
- **Path:** engine/patterns/sliding-head-bandsaw-mouse-drag-tempo-minigame.md
- **Type:** C
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #132, #043
- **Tags:** unity, minigame, mouse-drag, plankmaker, tempo

**Brief:** 4th minigame mechanic for TT (alongside zone-click, circular-cursor needle, hold-needle): mouse drag tempo for PlankMaker. Player drags mouse to push bandsaw head along log; tempo accuracy determines output count.

**Details:**
- Visual: sliding bandsaw head with blade, static log on tracks
- Input: player drags mouse left→right at consistent tempo
- Optimal range: 1.5-2.5 cm/s drag speed → PERFECT
- Tempo measurement: average velocity over last 0.5s window
- Cycles: multi-cycle minigame for thicker logs (WoodTypeData.cuttingCycles = 1/2/3 per species tier)
- Output per cycle: bad=1 plank, good=2, perfect=3 (quantity-not-quality — see #043)
- Bark per cycle by machine tier: poor=6, basic=4, premium=2

**Why it matters:** Variety in minigames prevents repetition fatigue. Mouse drag tempo is the 4th distinct mechanic — adds rhythm/feel category to TT's mechanic catalog.

#### #045 ★ MaterialPropertyBlock for runtime color variants
- **Slug:** material-property-block-runtime-color-variants
- **Path:** engine/patterns/material-property-block-runtime-color-variants.md
- **Type:** A
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #003, #097
- **Tags:** unity, materials, runtime, color, performance, mpb

**Brief:** Apply runtime color variants to shared material via MaterialPropertyBlock — zero material duplication, zero shader recompilation, zero extra VRAM.

**Details:**
- Problem: 8 NPC car color variants with naive approach = 8 duplicate materials per car part
- Solution: shared material, runtime MPB sets `_BaseColor` per instance
- Code pattern:
  ```csharp
  MaterialPropertyBlock mpb = new MaterialPropertyBlock();
  mpb.SetColor("_BaseColor", carColor);
  bodyRenderer.SetPropertyBlock(mpb);
  ```
- Caveat: MPB doesn't survive scene reload — must re-apply in Start
- Save/load: persist color as Color field on `ISaveable`, re-apply on load
- Convention: separate body Renderer (`Mat_Body`) from trim/glass/wheels (`Mat_Trim`, `Mat_Glass`) so only body color varies
- NPC cars: car color stored in CarVariantData SO (8 colors: white, red, blue, green, gray, yellow, brown, black)

**Why it matters:** 8 cars × 4 materials = 32 unique materials with naive approach. MPB: 1 shared material set, 8 runtime variants, identical performance, 1/32 the VRAM.

#### #047 Quest highlight pattern
- **Slug:** quest-highlight-pattern
- **Path:** engine/patterns/quest-highlight-pattern.md
- **Type:** A
- **Template:** pattern
- **Sources:** 03ec0ede-bc12
- **Related:** #048, #117
- **Tags:** unity, quest, highlight, ui, tutorial

**Brief:** Tutorial quests highlight target objects (outline/glow) on quest start, unhighlight on completion. Highlight uses quest-flag on the object — same object can be highlighted ONLY during its specific quest, not during subsequent gameplay.

**Details:**
- Highlight mechanism: outline shader on object + animated emissive
- Quest-flag pattern: `obj.questHighlightActive = true` when current quest targets this object
- Quest progression: QuestManager.OnQuestStart(questId) → loop through objects tagged for that quest → set flag
- On quest completion: unset flag → outline disappears
- Critical: flag prevents re-highlight in later gameplay (e.g., specific tree highlighted only during "Pierwsza wycinka" quest, not when player revisits the same tree later)
- Multi-step quests (see #048): highlight changes per step

**Why it matters:** Player needs clear "what now" cue during tutorial; same cue would be annoying in late game. Quest-flag mechanic threads the needle.

#### #052 River mesh 3D semi-ellipse cross-section formula
- **Slug:** river-mesh-semi-ellipse-cross-section
- **Path:** engine/patterns/river-mesh-semi-ellipse-cross-section.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6967a837-dc86
- **Related:** #051, #053
- **Tags:** unity, blender, mesh, river, water, terrain-blending

**Brief:** River geometry in TT uses semi-elliptical cross-section (9 verts per section, 113 sections at 5m spacing). Formula: `z = bank_z - depth*(1 - sqrt(1 - t²))` where t is normalized distance from centerline.

**Details:**
- Cross-section verts: -210 to +350 X-axis range, 9 verts per section across width
- Width per section: 3m (3D semi-ellipse, ~2.7m visible concavity)
- Depth at centerline: 2-5m (varies along course)
- Formula per vertex:
  ```python
  t = abs(distance_from_centerline / half_width)  # 0 at center, 1 at bank
  z = bank_z - depth * (1 - math.sqrt(1 - t*t))  # convex from above
  ```
- Centerline tracking: constraint max 6m change per 5m step (prevents discontinuity)
- Edge tucking: bank vertices = `min(z_ellipse, terrain_z - 0.15)` → ensures river edges sink below terrain, hides seam
- Generated via Blender Python script, exported as River.fbx (1017v, 896 faces → 2246v after Unity import)
- Used by `Custom/LowPolyWater` shader for surface animation (see #051)

**Why it matters:** "Why does my water look like a flat plane?" — because it IS a flat plane. Semi-ellipse cross-section + tucked edges = water that looks like it belongs in a riverbed.

#### #056 4-phase weighted smoothstep day/night transition
- **Slug:** four-phase-weighted-smoothstep-day-night
- **Path:** engine/patterns/four-phase-weighted-smoothstep-day-night.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #055, #057, #054
- **Tags:** unity, urp, shader, day-night, smoothstep, weighted

**Brief:** Day/night transitions use 4 weighted phases (night/dawn/day/sunset) with smoothstep overlapping bands. Color is sum of `noon*dayFactor + dawn*dawnFactor + dusk*duskFactor + night*nightFactor` — eliminates hard cuts.

**Details:**
- Time t ∈ [0, 1] represents 24-hour day cycle (cycleDurationMinutes = 2)
- 4 phases with overlapping smoothstep ranges:
  - night: t ∈ [0.88, 0.12] (wraps midnight)
  - dawn: t ∈ [0.12, 0.35]
  - day: t ∈ [0.25, 0.80]
  - dusk: t ∈ [0.70, 0.90]
- Each factor: `smoothstep(start, peak, t) * smoothstep(peak, end, t)`
- Final color: weighted sum, each color contributes by its factor
- Gameplay-first constraint: night minimum 55% intensity (NEVER fully black — players need to see)
- Skybox shader pushes `_TimeOfDay` per-frame, blends sky gradients
- Ambient color tracks time, NOT sun intensity (always proper ambient regardless of rotation — see #054 anti-pattern)

**Why it matters:** Without phase weighting, transitions have visible hard cuts. With it, time of day feels smooth and natural.

#### #057 Procedural skybox sun/moon trick
- **Slug:** procedural-skybox-sun-moon-trick
- **Path:** engine/patterns/procedural-skybox-sun-moon-trick.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #055, #056
- **Tags:** unity, urp, shader, skybox, sun-moon, procedural

**Brief:** Procedural skybox renders both sun (bright disc + halo) and moon (opposite-side disc) using single `_SunDirection` uniform. At night, shader sets `_SunDirection = -moonDir` so moon appears on east/west arc analogously.

**Details:**
- Skybox uniforms set per-frame by `DayNightCycle.cs`: `_TimeOfDay` (0-1), `_SunDirection` (Vector3)
- Day phase: `_SunDirection` = actual sun direction
- Night phase: `_SunDirection = -moonDir` → shader draws "sun" disc where moon actually is
- Sun rendering: `smoothstep(thresh, 1, dot(viewDir, _SunDirection))` for disc + `pow(dot, 32)` for halo
- Moon rendering: enabled only at night (faded by `_TimeOfDay`)
- Stars: procedural grid hash (~3% cell density), sinusoidal twinkle, only above horizon (`viewDir.y > 0`) and at night
- Both sun and moon travel same east-west arc (same `sunSkyboxYAngle`) — Minecraft-style

**Why it matters:** Single shader, two celestial bodies, zero extra GPU cost. Trick relies on understanding sun + moon are mathematically symmetric for day/night.

#### #060 Custom Editor pattern for generators
- **Slug:** custom-editor-pattern-for-generators
- **Path:** engine/patterns/custom-editor-pattern-for-generators.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #101, #105
- **Tags:** unity, editor, custom-editor, generator, terrain, ui

**Brief:** Generators (terrain, roads) get Custom Editor with prominent action button + preset row + export-as-asset button — designer can iterate without writing code.

**Details:**
- Inspector layout via OnInspectorGUI:
  - DrawDefaultInspector() (auto-fields)
  - Large green "🌲 GENERUJ TEREN" button (fontSize 16, bold, height 40)
  - Preset row: `EditorGUILayout.BeginHorizontal()` → 3 preset buttons (Płaski/Łagodne wzgórza/Górski) that set parameter values
  - Blue "💾 Eksportuj mesh jako Asset" button — opens save panel, calls `AssetDatabase.CreateAsset`
- Preset method example:
  ```csharp
  if (GUILayout.Button("Łagodne wzgórza")) {
      gen.maxHeight = 4;
      gen.noiseScale = 0.025f;
      gen.octaves = 3;
      EditorUtility.SetDirty(gen);
  }
  ```
- Wrap in `#if UNITY_EDITOR` directive
- Save: `EditorUtility.SaveFilePanelInProject(...)` then `AssetDatabase.CreateAsset(mesh, path)`

**Why it matters:** Lets Hunter iterate terrain shape via 3 button clicks instead of typing parameter values. Difference between "iteration is fun" and "iteration is tedious."

#### #063 ToolViewmodel pattern (child of camera)
- **Slug:** tool-viewmodel-child-of-camera-pattern
- **Path:** engine/patterns/tool-viewmodel-child-of-camera-pattern.md
- **Type:** A
- **Template:** pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** #061, #115
- **Tags:** unity, fps, viewmodel, camera, tool

**Brief:** FPS-style tool hold (axe, shovel) achieved by parenting the tool model to `PlayerCamera`. Tool moves with camera, no per-frame transform follow code needed.

**Details:**
- Hierarchy: `Player → PlayerCamera → ToolViewmodel` (parent chain)
- ToolViewmodel position/rotation set once relative to camera (right side, slightly below, angled inward)
- Auto-follows camera rotation/position (Unity transform inheritance)
- Tool swap: `ToolWheel` selects tool → `ToolViewmodel.Show(ToolType)` enables matching child mesh
- Mesh per tool: child GameObjects, only one active at a time
- Caveat: tool clips through walls (FPS standard issue) — leave alone for low-poly aesthetic, or add layer-based render queue (advanced)

**Why it matters:** "How do I make the tool move with the camera" — 5-minute solution. Parent it. Done.

#### #064 AudioManager + Mixer architecture (5 channels + 10-source pool)
- **Slug:** audio-manager-mixer-architecture
- **Path:** engine/patterns/audio-manager-mixer-architecture.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a, 6967a837-dc86
- **Related:** #065, #088, #117
- **Tags:** unity, audio, audio-mixer, singleton, performance

**Brief:** Single AudioManager (singleton, ISaveable for volume) with Audio Mixer routing 5 channels (Master/Music/SFX/Ambient/UI). Pool of 10 AudioSources for SFX reuse, crossfade for music transitions.

**Details:**
- Audio Mixer setup: Master → {Music, SFX, Ambient, UI} (sub-groups)
- AudioManager.Play(clip, AudioCategory) routes to correct group
- SFX uses pool of 10 AudioSources (rotate when all busy, oldest interrupted)
- Music crossfade: 2 sources alternating, 1s fade in / 1s fade out
- Spatial Blend: SFX/Ambient = 1.0 (3D positional), Music/UI = 0.0 (2D)
- Logarithmic rolloff: Min Distance 1m, Max Distance 30-50m (per AudioConfigSO)
- Volume saved/loaded as ISaveable: master, music, sfx, ambient, ui (5 floats 0-1)
- Mixer snapshots per Game State (see #088): Playing=full, Paused=muted SFX/Ambient, Menu=muted gameplay

**Why it matters:** Without a single manager, every audio call leaks AudioSources. With it, audio "just works" and respects player volume preferences.

#### #066 ★ ServiceLocator + GameEventSO + ISaveable + Singleton parallel pattern
- **Slug:** parallel-architecture-pattern
- **Path:** engine/patterns/parallel-architecture-pattern.md
- **Type:** D
- **Template:** pattern
- **Sources:** 6967a837-dc86, e55dbfd6-bb50
- **Related:** #067, #068, #037
- **Tags:** unity, architecture, service-locator, scriptableobject, isaveable, migration

**Brief:** TT runs 4 architecture systems in parallel: ServiceLocator (dependency lookup), GameEventSO (decoupled messaging), ISaveable (persistence), Singleton (legacy/compatibility). Migration is gradual, not big-bang.

**Details:**
- **ServiceLocator**: `Services.Register<T>(this)` in Awake, `Services.Get<T>()` anywhere. Replaces `FindObjectOfType`.
- **GameEventSO**: ScriptableObject event channels in `Assets/ScriptableObjects/Events/`. Raise/Register/Unregister pattern (see #067).
- **ISaveable**: contract for persistence (see #068). Auto-registered in SaveManager.
- **Singleton**: legacy pattern (`Instance` static), still works during migration.
- Parallel = all 4 coexist temporarily. Singleton calls work alongside ServiceLocator calls. Migration is feature-by-feature.
- Rule: NEW code uses ServiceLocator + GameEventSO. OLD code migrated when touched ("boy scout rule").
- 10 manager singletons currently: ServiceLocator, GameEventSO, GameStateMachine, InputManager, SaveManager, AudioManager, SceneTransitionManager, LocalizationManager, TimeManager, PhysicsOptimizer
- All in `--- MANAGERS ---` section of scene hierarchy

**Why it matters:** Big-bang refactor = months of broken builds. Parallel architecture = ship the demo, migrate incrementally. This is the actual TT architecture, codified.

#### #067 ★ GameEventSO ScriptableObject event channel
- **Slug:** game-event-so-event-channel
- **Path:** engine/patterns/game-event-so-event-channel.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6967a837-dc86, a0ce5046-a69a
- **Related:** #066, #076, #068
- **Tags:** unity, scriptableobject, events, messaging, decoupled

**Brief:** GameEventSO = ScriptableObject acting as event channel. Raise/Register/Unregister methods. Decoupled messaging between systems — no direct references between managers.

**Details:**
- Base class:
  ```csharp
  [CreateAssetMenu(menuName = "Timber Tycoon/Events/Game Event")]
  public class GameEventSO : ScriptableObject {
      private List<Action> listeners = new();
      public void Register(Action listener) => listeners.Add(listener);
      public void Unregister(Action listener) => listeners.Remove(listener);
      public void Raise() => listeners.ForEach(l => l.Invoke());
  }
  ```
- Generic variant for typed payloads: `GameEventSO<T>` (e.g., `OnTreeCut: GameEventSO<TreeTypeData>`)
- 8 event channels currently in `Assets/ScriptableObjects/Events/`: OnMoneyChanged, OnTreeCut, OnLogPickup, OnSale, OnDayChanged, OnQuestComplete, OnUpgradeUnlocked, OnPlayerInteract
- Subscriber pattern: register in OnEnable, unregister in OnDisable (mandatory to prevent leaks)
- VFX/Audio hook here via VFXTrigger.cs / AudioManager (see #076)

**Why it matters:** Without event channels, every manager directly references every other manager → tangled web. Event channels = clean separation, easy testing, swappable systems.

#### #068 ★ ISaveable contract
- **Slug:** isaveable-contract
- **Path:** engine/patterns/isaveable-contract.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6967a837-dc86, e55dbfd6-bb50
- **Related:** #066, #021
- **Tags:** unity, save-system, isaveable, persistence, json

**Brief:** ISaveable interface contract: `GetSaveKey()` + `GetSaveData()` + `LoadSaveData()`. SaveManager auto-registers all ISaveable on scene load, persists to JSON.

**Details:**
- Interface:
  ```csharp
  public interface ISaveable {
      string GetSaveKey();              // unique per component, e.g. "VehicleStorage_Car1"
      string GetSaveData();             // serialize state to JSON string
      void LoadSaveData(string json);   // deserialize from JSON
  }
  ```
- SaveManager.Start: `FindObjectsOfType<MonoBehaviour>().OfType<ISaveable>()` → register each
- Save: SaveManager iterates registered ISaveables, builds combined JSON, writes to slot file
- Load: read JSON, dispatch to each ISaveable by key
- Auto-save every 5 minutes, 3 slots
- Implement on every stateful component: VehicleController (position), VehicleStorage (logs), QuestManager (phase + flags), EconomyManager (money), ToolManager (unlocked + selected)
- Awake-init for ISaveable with dependencies (see #021)

**Why it matters:** Single interface = single save format = single load code path. Every new system gets save/load for free by implementing 3 methods.

#### #069 GameStateMachine pattern
- **Slug:** game-state-machine-pattern
- **Path:** engine/patterns/game-state-machine-pattern.md
- **Type:** C
- **Template:** pattern
- **Sources:** 6967a837-dc86, a0ce5046-a69a
- **Related:** #117, #066
- **Tags:** unity, state-machine, game-state, input-gating

**Brief:** GameStateMachine controls high-level game state (Boot/Splash/MainMenu/Loading/Playing/Paused/Cutscene). State gates input — PlayerController and PlayerInteraction check state before processing.

**Details:**
- Enum: `GameState { Boot, Splash, MainMenu, Loading, Playing, Paused, Cutscene }`
- Transitions: Boot → Splash → MainMenu → Loading → Playing → (Paused | Cutscene) → Playing
- During Paused: `Time.timeScale = 0`, only UI input processed
- During Cutscene: player input blocked, camera controlled by Cinemachine or sequence
- During Playing: full input, all systems active
- Gating in code: `if (GameStateMachine.Current != GameState.Playing) return;`
- ISaveable: persist game state in case of crash recovery
- Combine with #117 (canMove flag) for finer-grained input control

**Why it matters:** Without explicit state, "pause menu still lets player move" and "cutscene gets interrupted by player input" are recurring bugs. State machine = clear contract.

#### #075 VFX performance budget
- **Slug:** vfx-performance-budget
- **Path:** engine/patterns/vfx-performance-budget.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #076, #074
- **Tags:** unity, vfx, particle-system, performance, budget

**Brief:** Scene-wide VFX budget for TT: max 500 particles total across all systems, max 10 simultaneously active ParticleSystems. Enforced via VFXManager throttling.

**Details:**
- 500 particles total: prevents over-emission destroying frame rate on lower-end hardware
- 10 active systems: keep particle-system overhead low
- Per VFX category: emission rate × lifetime × max systems ≤ category quota
- Enforcement: VFXManager (singleton) tracks active VFX, denies new spawn if over budget (queues or skips)
- Per-VFX prefab: configure Max Particles in inspector to safe value
- Naming: `vfx_kategoria_opis` in `Assets/VFX/`
- Trigger via GameEventSO (see #076)

**Why it matters:** Without budget, "subtle dust effect" spawns 5000 particles when 50 trees fall simultaneously → 30fps. With budget, 60fps locked.

#### #076 VFX trigger pattern via GameEventSO + VFXTrigger.cs
- **Slug:** vfx-trigger-pattern-via-game-event
- **Path:** engine/patterns/vfx-trigger-pattern.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #067, #075
- **Tags:** unity, vfx, game-event, decoupled, trigger

**Brief:** VFX prefabs are spawned by `VFXTrigger.cs` components that listen to GameEventSO. ChoppableTree raises `OnTreeCut`, VFXTrigger spawns sawdust at hit position. Decoupled — gameplay code doesn't reference VFX prefabs.

**Details:**
- VFXTrigger.cs subscribes to specific GameEventSO in OnEnable, unsubscribes in OnDisable
- Configure in Inspector: event channel + VFX prefab to spawn + offset + lifetime
- Example: VFXTrigger listens to OnTreeCut, spawns `vfx_sawdust.prefab` at event position
- Sample VFX catalog (planned, some wycofane — see #074):
  - sawdust (OnTreeCut) — WYCOFANE
  - kurz (OnTreeFall) — WYCOFANE  
  - liście (OnTreeFall seasonal) — WYCOFANE
  - spaliny (Vehicle moving)
  - iskry (Sawmill active)
  - dym komina (Sawmill active)
  - splash (Vehicle enters river)
  - złote monety (OnSale)
- VFXTrigger checks VFXManager budget before spawn (see #075)

**Why it matters:** Adding new VFX = adding a Trigger component, no gameplay code changes. Decoupled and budget-respecting.


---

### BATCH 3 — engine/patterns part 2 (entries #048-#075)

#### #077 Mountains hierarchy — front ring + backdrop double-sided
- **Slug:** mountains-hierarchy-front-and-backdrop
- **Path:** engine/patterns/mountains-hierarchy-front-and-backdrop.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6967a837-dc86
- **Related:** #078, #056
- **Tags:** unity, blender, mountains, backdrop, double-sided, low-poly

**Brief:** Mountains in TT are two FBX files: Mountains.fbx (front ring, single-sided, 40-90m peaks) + Mountains_Backdrop.fbx (back ring, double-sided, 100-151m, overlap 10m). Prevents horizon gap.

**Details:**
- Front Mountains.fbx: 8.2k verts, low-poly peaks 40-90m height, pierścień dookoła mapy, single-sided
- Backdrop Mountains_Backdrop.fbx: 2100 verts, 100-151m height, 48 segmentów × 4 pierścienie radialne, 70m grubości, **double-sided** (mesh visible from both sides)
- Overlap: front ring's outer edge intersects backdrop's inner edge by 10m → no visible seam
- Backdrop is double-sided because camera occasionally sees it from inside (mountain pass, high elevation viewpoint)
- Material: `Mat_Backdrop` uses `Custom/VertexColorLit` (Brightness=0.5 → desaturated/distant feel)
- Inner mountains v2: 30 individual mountains, 7 profile variants, 5 shader layers via `Custom/MountainLayered`

**Why it matters:** Without backdrop, the map ends at a hard horizon line. With backdrop, distant mountains create depth and contain the player visually.

#### #079 Cliff+Waterfall hidden cave pattern
- **Slug:** cliff-waterfall-hidden-cave
- **Path:** engine/patterns/cliff-waterfall-hidden-cave.md
- **Type:** C
- **Template:** pattern
- **Sources:** 6967a837-dc86
- **Related:** #078, #053
- **Tags:** unity, level-design, secret, waterfall, cave, player-discovery

**Brief:** Waterfall (no collider, transparent) hangs over cliff face, hiding cave entrance behind it (mesh collider on cliff). Player walks through waterfall to discover cave.

**Details:**
- Cliff_Waterfall.fbx (640v): trapezoidalny profil (26m góra → 55m dół, ~20° skośne boki), jaskinia 12×8×17.5m kopuła, vertex colors avg (0.43, 0.39, 0.32) linear
- Waterfall.fbx (140v): odwrócone L (8m poziomo + 45m pionowy spadek), 14m szerokość, vertex colors biała piana góra/dół + teal woda środek
- Cliff has MeshCollider (player walks on rock, can't pass through)
- Waterfall has NO collider (player walks through water)
- Cave entrance inside cliff, hidden by waterfall from outside view
- Player discovery: approach cliff, walk into waterfall, find hidden cave
- Both pieces positioned manually by designer (hardcoded — see #080)

**Why it matters:** Free "exploration moment" with minimal designer effort. Player feels rewarded for curiosity. Pattern reusable for other hidden areas.

#### #088 Audio Mixer snapshots per Game State
- **Slug:** audio-mixer-snapshots-per-game-state
- **Path:** engine/patterns/audio-mixer-snapshots-per-game-state.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #064, #069
- **Tags:** unity, audio, mixer, snapshots, game-state

**Brief:** Audio Mixer snapshots define volume mix per game state. GameStateMachine triggers snapshot transitions: Playing=full, Paused=muted SFX/Ambient + keep UI, Menu=muted gameplay + boost music.

**Details:**
- Create snapshots in Audio Mixer asset: Playing, Paused, Menu (right-click mixer → Add Snapshot)
- Per snapshot: set group volumes (e.g., Paused snapshot: SFX -40dB, Ambient -40dB, Music -10dB, UI 0dB)
- Trigger transitions from code: `mixer.FindSnapshot("Paused").TransitionTo(0.3f)` (0.3s fade)
- Hook to GameStateMachine: subscribe to state change, transition matching snapshot
- Use case: pause menu — SFX immediately dampened, UI clicks audible at normal volume
- Use case: cutscene — pre-defined dramatic snapshot (music up, ambient down)

**Why it matters:** Without snapshots, pause feels "loud and chaotic" (SFX trailing off slowly). With snapshots, pause feels intentional and clean.

#### #089 Footstep raycast pattern
- **Slug:** footstep-raycast-surface-detection
- **Path:** engine/patterns/footstep-raycast-surface-detection.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #064
- **Tags:** unity, audio, footstep, raycast, terrain, sfx

**Brief:** Footstep SFX detection: raycast down from player feet → identify surface layer → play matching SFX from lookup table (grass/dirt/wood/stone/water).

**Details:**
- On footstep event (animation event or distance-traveled trigger):
  - `Physics.Raycast(transform.position + Vector3.up * 0.1f, Vector3.down, hit, 0.5f, terrainLayerMask)`
  - hit.collider.gameObject.layer → look up matching SFX (e.g., layer "Wood" → footstep_wood.wav)
- Lookup table: `Dictionary<int, AudioClip[]>` in FootstepSO
- Multiple clips per surface, random selection for variety (4-6 clips each)
- For TT: Grass (Default + Trees), Dirt (road), Wood (sawmill floor + bridge), Stone (cliff path)
- Water: special — check `WaterZone` collider, play splash + wade audio
- Volume: scale with movement speed (sprint louder than walk)

**Why it matters:** Single footstep sound across all surfaces = "video game-y." Surface-aware footsteps = immersion jump.

#### #090 Audio occlusion pattern (LPF + volume)
- **Slug:** audio-occlusion-lpf-volume
- **Path:** engine/patterns/audio-occlusion-lpf-volume.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #064, #091
- **Tags:** unity, audio, occlusion, low-pass-filter, spatial

**Brief:** Simple audio occlusion: raycast from listener to audio source. If obstructed by colliders, apply Low Pass Filter + volume reduction. Cheaper than full HRTF occlusion.

**Details:**
- On AudioSource Update:
  - `Physics.Raycast(listener.position, (source.position - listener.position).normalized, hit, distance, obstructionLayerMask)`
  - If hit collider exists AND not the audio source itself: occluded
- Occluded state: enable LowPassFilter component on AudioSource (cutoff ~800Hz)
- Volume: reduce by 30-50% (depends on perceived material — concrete vs. fabric)
- Smooth transitions: lerp filter cutoff over 0.2s when state changes
- Performance: don't update every frame — every 0.1s sufficient
- Skip for 2D sounds (UI/music) and short SFX (< 0.5s — won't notice)

**Why it matters:** "Sawmill machine sounds the same from inside and outside the building" = breaks immersion. Cheap occlusion fixes it.

#### #091 AudioReverbZone per environment
- **Slug:** audio-reverb-zone-per-environment
- **Path:** engine/patterns/audio-reverb-zone-per-environment.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #064, #092
- **Tags:** unity, audio, reverb, environment, ambient

**Brief:** AudioReverbZone components in different environments give location-specific acoustics: building interior = reverb, forest = light reverb, open field = dry.

**Details:**
- Place AudioReverbZone on environment boundaries: inside building (reverb preset = SmallRoom or Hallway), forest (Forest preset, light), open (Off/Dry)
- AudioReverbZone has Inner Radius (full effect) and Outer Radius (fade-out)
- Listener inside zone → reverb applied to all AudioSources passing through
- Overlap zones for smooth transitions (e.g., near building edge gets blend of indoor + outdoor)
- Per TT environment: Sawmill interior (SmallRoom), Forest (Forest light), Bridge (Concrete sparse), River cave (Cave preset)

**Why it matters:** "Footstep in sawmill sounds same as footstep in forest" = immersion broken. Per-environment reverb = each location feels distinct.

#### #092 Ambient crossfade zone-based pattern
- **Slug:** ambient-crossfade-zone-based
- **Path:** engine/patterns/ambient-crossfade-zone-based.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a
- **Related:** #064, #091
- **Tags:** unity, audio, ambient, crossfade, zones

**Brief:** Ambient soundscape switches based on player zone via crossfade. Forest ambient (birds, wind in trees) fades out as player enters sawmill (machine hum), then dirt road (vehicle distant).

**Details:**
- AmbientManager (singleton) tracks current zone, plays matching loop
- Zone detection via trigger colliders or distance to zone center
- Crossfade: 2 AudioSources, fade old out 1s while new fades in 1s
- Per zone: AmbientZoneSO with clip + volume + reverb override
- Zones in TT: Forest (default), Sawmill interior, Road, River, City (planned)
- Transitions smooth, never silent (always 1 ambient layer at min volume)

**Why it matters:** Without ambient, areas feel sterile. With per-zone ambient + crossfade, transitions between areas feel meaningful.

#### #093+#094+#095 ★ Typography & accessibility stack [MERGED]
- **Slug:** typography-accessibility-stack
- **Path:** engine/patterns/typography-accessibility-stack.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a, e55dbfd6-bb50
- **Related:** #87
- **Tags:** unity, ui, typography, accessibility, textmeshpro, font, sdf
- **Dedup:** Merger of #093 (TextMeshPro rule), #094 (style guide), #095 (dynamic sizing) — single comprehensive pattern doc

**Brief:** TT typography stack: TextMeshPro mandatory (NIGDY legacy Text), Polish character font (Noto Sans/Inter/Roboto) with SDF atlas, defined size scale + color palette, accessibility scaling × 1.0/1.25/1.5.

**Details:**

**Part 1 — TextMeshPro rule (#093):**
- NEVER use Unity legacy `Text` component — TextMeshPro mandatory
- Polish character support requires fonts with full extended Latin set: Noto Sans, Inter, Roboto recommended (TT uses Nunito from Google Fonts, OFL license)
- Generate SDF atlas: TMP Font Asset Creator → assign font file → set Atlas Resolution 2048×2048 → Sampling Point Size 90 → Generate Font Atlas
- Fallback font chain for special characters (CJK if future localization): assigned in TMP Settings

**Part 2 — Style guide (#094):**
- Font size hierarchy:
  - Header: 32pt
  - Subheader: 24pt
  - Body: 18pt
  - Caption: 14pt
  - Tooltip: 16pt
- Color palette (define in `UIStyleGuideSO`):
  - Primary text: white
  - Secondary text: light gray (#B5B5B5)
  - Accent: brand color
  - Positive: green (#5BAA47)
  - Negative: red (#C75F50)
  - Disabled: dark gray (#666666)

**Part 3 — Accessibility scaling (#095):**
- Dynamic font size: `AccessibilityManager.FontSizeMultiplier` (0.8 / 1.0 / 1.3)
- All TMP_Text components register via `AccessibilityManager.RegisterScalableText(text)`
- On setting change: `RefreshAllFontSizes()` iterates registered, applies multiplier × base size
- SDF rendering ensures crisp output at all sizes — outline/shadow in SDF material
- ISaveable: persist setting

**Why it matters:** Accessibility isn't optional (Steam Deck players, low-vision users, multiple languages). Stack designed once = scales for all UI without per-screen retrofit.

#### #097 Shared mesh + materials reference pattern
- **Slug:** shared-mesh-and-materials-reference
- **Path:** engine/patterns/shared-mesh-and-materials-reference.md
- **Type:** A
- **Template:** pattern
- **Sources:** e12ff52d-926c
- **Related:** #045, #098
- **Tags:** unity, prefabs, references, placeholders, debug

**Brief:** Visual placeholders (e.g., PelletRack_Placeholder) reference SAME mesh + materials as final asset (BagRack) — no asset duplication. Faster iteration, eliminates "placeholder vs. final" drift.

**Details:**
- Bad pattern: duplicate mesh + materials for placeholder, swap on finalization → manual sync required, drift inevitable
- Good pattern: placeholder GameObject's MeshFilter / MeshRenderer reference SAME mesh asset and SAME materials as final
- Diff between placeholder and final = transform, layer, components — NOT mesh/material assets
- Update final asset → placeholder updates automatically (visual parity)
- Workflow during dev: place placeholder, configure StorageRack component, iterate. Replace with final prefab when ready (drag prefab into scene, copy transform/scale).

**Why it matters:** "Placeholder looks different than final" is the #1 reason demos look unfinished. Same asset references = identical rendering, zero drift.

#### #098 Rack visual fill alignment pattern
- **Slug:** rack-visual-fill-alignment
- **Path:** engine/patterns/rack-visual-fill-alignment.md
- **Type:** A
- **Template:** pattern
- **Sources:** e12ff52d-926c
- **Related:** #096+119, #097
- **Tags:** unity, prefabs, racks, fill-visualization, alignment

**Brief:** Storage racks visualize fill level via child `fillLevel1/2/3` transforms aligned to shelf positions in the final rack model.

**Details:**
- Rack model has Shelf_1, Shelf_2, Shelf_3 at fixed Y positions (e.g., 0.40, 1.00, 1.60)
- StorageRack component has fillLevel1, fillLevel2, fillLevel3 child transforms
- Each fillLevel positioned at corresponding Shelf Y (so bags rest on shelves visually)
- At runtime, instantiate visual product prefabs as children of fillLevel transforms (e.g., 3 BagOfPellet prefabs on fillLevel1 = first shelf full)
- Capacity logic: count items in fillLevel transforms vs maxCapacity (9 for 3-shelf rack)
- Per-product visual representation defined in StorageFamilySO

**Why it matters:** "Bags float in air instead of sitting on shelf" — alignment pattern fixes it. Pre-positioned transforms make placement deterministic.

#### #102 Catmull-Rom spline + quad strip mesh for rendered roads
- **Slug:** catmull-rom-spline-road-mesh
- **Path:** engine/patterns/catmull-rom-spline-road-mesh.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #103, #104
- **Tags:** unity, road, mesh-generation, spline, catmull-rom

**Brief:** Roads generated from waypoints via Catmull-Rom spline → quad strip mesh + vertex colors + Perlin variation per road type.

**Details:**
- Waypoints list defines road path (added via RoadTool editor — see #105)
- Catmull-Rom: smooth spline through waypoints, configurable `splineResolution` (default 10 segments between waypoints)
- Quad strip mesh: for each spline segment, generate quad (4 verts) perpendicular to spline direction at `roadWidth`
- Vertex colors per type: Dirt brown, Asphalt dark gray, Gravel light gray, Forest path, Sidewalk light
- Perlin noise per vertex for organic color variation (subtle)
- Shader: `Custom/LowPolyVertexColor` (same as terrain)
- 5 road types in TT, each with preset color + default width

**Why it matters:** Hand-modeled roads in Blender are tedious. Catmull-Rom in Unity = paint roads in Scene View with editor tool, iteration in seconds.

#### #103 FlattenTerrainUnderRoad smoothstep blend
- **Slug:** flatten-terrain-under-road-smoothstep
- **Path:** engine/patterns/flatten-terrain-under-road.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #102
- **Tags:** unity, road, terrain, mesh-modification, smoothstep

**Brief:** Roads need flat terrain underneath. `FlattenTerrainUnderRoad()` modifies terrain mesh vertices with smoothstep blend zone — graceful edge transition, not hard step.

**Details:**
- For each terrain vertex within `roadWidth + blendZone` of spline:
  - Compute distance from spline centerline
  - `t = smoothstep(roadWidth, roadWidth + blendZone, distance)` — 0 at road, 1 at far edge
  - Lerp Y from `roadY` (flat) to `terrainY` (natural) by t
- Result: road is flat, surrounding terrain gracefully transitions
- Blend zone width: 1-2x road width (e.g., 3m road → 4m blend zone each side)
- Modify terrain mesh in-place, then refresh MeshCollider
- Save modified terrain mesh as asset to persist (otherwise rebuild loses changes)

**Why it matters:** Hard terrain edges = visible seam where road meets ground. Smoothstep blend = roads look like they belong on the terrain.

#### #104 MeshCollider on roads = stackable
- **Slug:** mesh-collider-on-roads-stackable
- **Path:** engine/patterns/mesh-collider-on-roads-stackable.md
- **Type:** A
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #102, #103
- **Tags:** unity, road, collider, stacking, raycast

**Brief:** Roads have MeshColliders. Allows new roads to raycast onto existing roads (stack sidewalk on asphalt, cross asphalt with dirt path).

**Details:**
- After generating road mesh, assign to MeshCollider component
- New road's RoadTool uses raycast on `Default | Road | Terrain` layers when clicking to place waypoints
- Hit on existing road → waypoint placed on road surface (Y matches road height, not terrain)
- Enables crossings, intersections, sidewalks-on-asphalt without manual height adjustment
- Performance: MeshCollider is moderately expensive — keep road meshes simple (don't subdivide gratuitously)

**Why it matters:** Without stacking, sidewalk on asphalt = manually edit Y values. With stacking, it just works.

#### #108 ★ ScriptableObject runtime injection pattern
- **Slug:** scriptable-object-runtime-injection
- **Path:** engine/patterns/scriptable-object-runtime-injection.md
- **Type:** C
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #109, #111, #039
- **Tags:** unity, scriptableobject, runtime-injection, extension-by-data

**Brief:** Component reads SO data in Start() with null-check fallback to Inspector-set prefab fields. Enables "zero code changes per species/type" — add new content via new SO + models.

**Details:**
- Component has both: individual prefab fields AND a TreeTypeData SO field
- Start() pattern:
  ```csharp
  void Start() {
      if (treeTypeData != null) {
          // SO-driven path
          treeType = treeTypeData.treeType;
          stumpPrefab = treeTypeData.stumpPrefab;
          fallenTrunkPrefab = treeTypeData.fallenTrunkPrefab;
          // etc.
      }
      // else: use Inspector-set values (fallback)
  }
  ```
- Adding new species = create TreeTypeData SO + assign prefabs + drop on prefab. Zero code changes.
- See also: #109 (propagation chain), #111 (philosophy), sweep2-3 (race condition fix)
- Same pattern applies to WeaponData, ItemData, any "type variant" content

**Why it matters:** Content scaling. 4 tree species today, 12 tomorrow, same code. Pattern is fundamental to TT's data-driven design.

#### #109 SO propagation chain via parameter passing
- **Slug:** so-propagation-chain-via-parameters
- **Path:** engine/patterns/so-propagation-chain-via-parameters.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #108, #111
- **Tags:** unity, scriptableobject, propagation, parameter-passing

**Brief:** SO references propagate via parameter passing through spawn chains: ChoppableTree → DiggableStump → PlantingSpot. Each parent passes SO ref to child explicitly, not via singleton lookup.

**Details:**
- ChoppableTree.SpawnStump(): instantiates stump prefab, sets `stump.GetComponent<DiggableStump>().treeTypeData = this.treeTypeData`
- DiggableStump.SpawnPlantingSpot(): instantiates plantingSpot prefab, but does NOT pass treeTypeData (see #110 — PlantingSpot is universal)
- Sapling (PickupSapling) carries treeTypeData instead (see #110)
- Player plants sapling in PlantingSpot → PlantingSpot reads treeTypeData from sapling, passes to GrowingTree
- Why parameters not singletons: avoids global state, each spawned object owns its data
- Race condition caveat: use Init() method after Instantiate (see sweep2-3)

**Why it matters:** Singleton-based SO lookup = global state = bugs. Explicit propagation = clear data flow.

#### #114 Trunk fall physics config
- **Slug:** trunk-fall-physics-config
- **Path:** engine/patterns/trunk-fall-physics-config.md
- **Type:** C
- **Template:** pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** #112, #113
- **Tags:** unity, physics, rigidbody, tree-cutting, fallen-trunk

**Brief:** Fallen trunk physics config: Rigidbody mass=80, useGravity=true, isKinematic=false at spawn. Auto-stabilize to kinematic after 1.5s to prevent jitter.

**Details:**
- Spawn config:
  - Rigidbody.mass = 80 (heavy enough to behave like log, light enough to not destroy terrain)
  - Rigidbody.useGravity = true
  - Rigidbody.isKinematic = false (physics-driven during fall)
  - CapsuleCollider direction=0, radius=0.15, height=3.5 (from prefab, NOT overwritten — see #112)
- After 1.5s: switch isKinematic = true → trunk freezes in place, no further physics jitter
- Why: keeps physics simulation cost low (one log = one rigidbody for ~1.5s, then static)
- Multiple fallen trunks: don't compound interaction (kinematic → kinematic = no force exchange)
- Ignore collision with player temporarily (1s grace period — prevents trunk knocking player on spawn)

**Why it matters:** Without auto-stabilize, fallen trunks jiggle indefinitely. With it, world feels settled after each tree falls.

#### #115 ★ VehicleCamera third-person orbit (runtime attach/detach)
- **Slug:** vehicle-camera-runtime-attach-detach
- **Path:** engine/patterns/vehicle-camera-runtime-attach-detach.md
- **Type:** A
- **Template:** pattern
- **Sources:** 6d0820c1-8138
- **Related:** #063, #116
- **Tags:** unity, camera, third-person, orbit, runtime, vehicle

**Brief:** Third-person orbit camera for vehicles: mouse X/Y orbits, scroll zooms (4-15m), smooth Lerp follow. Component attached to player camera at runtime when entering vehicle, removed when exiting.

**Details:**
- VehicleCamera.cs (MonoBehaviour, not a separate Camera GameObject)
- On enter vehicle: `camera.gameObject.AddComponent<VehicleCamera>()` + initialize target
- VehicleCamera.Update():
  - Mouse X → yaw rotation around target
  - Mouse Y → pitch (clamped -45° to +85° to prevent flipping)
  - Scroll → zoom (target distance, 4-15m range)
  - Position = target + offset × distance, with smooth Lerp toward goal
- On exit vehicle: `Destroy(GetComponent<VehicleCamera>())` + reset camera to player viewmodel
- No separate camera GameObject = no state mismatch on enter/exit
- Critical: PlayerController + PlayerInteraction disabled during vehicle mode (see #116)

**Why it matters:** Separate camera GameObject leads to "camera stuck in wrong state" bugs. Runtime attach/detach = single source of truth (player's camera).

#### #116 Vehicle enter/exit choreography sequence
- **Slug:** vehicle-enter-exit-choreography
- **Path:** engine/patterns/vehicle-enter-exit-choreography.md
- **Type:** C
- **Template:** pattern
- **Sources:** 6d0820c1-8138
- **Related:** #115, #022
- **Tags:** unity, vehicle, enter, exit, choreography, sequence

**Brief:** Vehicle enter sequence: disable PlayerController + CharacterController + PlayerInteraction → parent player to vehicle → unparent camera + add VehicleCamera. Exit reverses, places player on vehicle's left side.

**Details:**
- Enter sequence (`VehicleInteractable.OnInteract`):
  1. `player.GetComponent<PlayerController>().enabled = false`
  2. `player.GetComponent<CharacterController>().enabled = false`
  3. `player.GetComponent<PlayerInteraction>().enabled = false`
  4. `player.transform.SetParent(vehicleRoot.transform)` (player hidden inside cabin)
  5. `camera.transform.SetParent(null)` (camera detaches from player)
  6. `camera.gameObject.AddComponent<VehicleCamera>().target = vehicleRoot.transform`
- Exit sequence (1s hold E):
  - Reverse: destroy VehicleCamera, re-parent camera to player, re-enable scripts
  - Place player: `player.transform.position = vehicle.position + vehicle.right * -1.5f + vehicle.up * 0.5f` (left side of vehicle, ground level)
  - Re-enable CharacterController AFTER position set (else CC overrides)
- Critical: order matters (disable scripts BEFORE re-parenting)

**Why it matters:** Without exact sequence, player gets stuck inside vehicle or camera floats away. Explicit choreography = reliable enter/exit every time.

#### #117 ★ Universal camera lock pattern (canMove flag)
- **Slug:** universal-camera-lock-canmove-flag
- **Path:** engine/patterns/universal-camera-lock-canmove-flag.md
- **Type:** A
- **Template:** pattern
- **Sources:** fda34b50-7e59
- **Related:** #041, #069, #115
- **Tags:** unity, camera, minigame, input-gating, player-controller

**Brief:** All TT minigames disable camera/movement via single flag: `PlayerController.canMove = false`. Universal — works for every minigame past, present, and future without per-minigame camera code.

**Details:**
- PlayerController has `public bool canMove = true;` field
- Update checks: `if (!canMove) return;` (no movement processing)
- Mouse look check: `if (!canMove) return;` (no rotation processing)
- Minigame start: `playerController.canMove = false`
- Minigame end: `playerController.canMove = true`
- Combined with #069 (GameStateMachine.Cutscene/Paused state) for double-safety
- New minigames automatically respect this flag — zero camera/input code needed in minigame logic
- For full camera position lock (not just rotation), combine with #041 (save/lerp/restore pattern)

**Why it matters:** Per-minigame camera disabling = N implementations of same logic, N places to forget the toggle. One flag = one place to maintain.

#### #118 StatisticsManager pattern
- **Slug:** statistics-manager-pattern
- **Path:** engine/patterns/statistics-manager-pattern.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50, a0ce5046-a69a
- **Related:** #066, #068
- **Tags:** unity, statistics, journal, isaveable, ui, events

**Brief:** StatisticsManager singleton: tracks 11 lifetime stats (trees cut, money earned, products made, distance traveled, etc.) + capped event journal (max 200). ISaveable. UI: 2 tabs (stats + journal).

**Details:**
- 11 stats tracked: trees cut, stumps removed, saplings planted, logs collected, products made, customers served, money earned, money spent, days played, kilometers driven, upgrades purchased
- Journal: capped queue of 200 events, oldest dropped on overflow
- Static events in gameplay code increment stats: `ChoppableTree.OnTreeCut(species)` → `StatisticsManager.IncrementTreesCut(species)`
- Statistics UI panel: 2 tabs (stats list + journal scroll), accessed from PauseMenuUI
- ISaveable: all stats + journal persist across sessions
- Used for achievement triggers, end-of-day summary, end-of-demo screen

**Why it matters:** Players love numbers about their gameplay. Stats manager is one component, hooks via existing events, near-zero ongoing cost.

#### #120 ★ StorageRackRegistry singleton + auto-rejestracja via OnEnable
- **Slug:** storage-rack-registry-auto-register
- **Path:** engine/patterns/storage-rack-registry-auto-register.md
- **Type:** A
- **Template:** pattern
- **Sources:** acb055d2-01aa
- **Related:** #096+119, #121, #122
- **Tags:** unity, storage, registry, singleton, on-enable

**Brief:** StorageRackRegistry singleton lookup-table by family enum. Racks auto-register via OnEnable (NOT manually wired in StorageManager). Lookup: `TryGetRackByFamily(StorageFamily.Plank) → StorageRack`.

**Details:**
- StorageRackRegistry (singleton): `Dictionary<StorageFamily, StorageRack> racks`
- StorageRack.OnEnable: `StorageRackRegistry.Instance.Register(family, this)`
- StorageRack.OnDisable: `StorageRackRegistry.Instance.Unregister(family)`
- Lookup: `if (StorageRackRegistry.Instance.TryGetRackByFamily(family, out var rack)) ...`
- StorageManager (separate singleton) uses Registry to route product storage (see #121)
- Adding new rack = drop GameObject with StorageRack component, set family. Zero StorageManager edits.
- Multiple racks of same family: pick the first registered (or extend to list if needed for capacity overflow)

**Why it matters:** Without auto-rejestracja, every new rack = "I forgot to wire StorageManager." With it, just drop and configure. Pure additive growth.


---

### BATCH 4 — engine/patterns part 3 + engine/anti-patterns (entries #121-#143)

#### #121 ★ GLOBAL_ROUTER pattern (StorageManager.AddToStorage)
- **Slug:** global-router-storage-pattern
- **Path:** engine/patterns/global-router-storage-pattern.md
- **Type:** C
- **Template:** pattern
- **Sources:** acb055d2-01aa
- **Related:** #120, #122, #096-119
- **Tags:** unity, storage, routing, decoupled, manager

**Brief:** `StorageManager.AddToStorage(ProductType, count)` routes products to correct StorageRack via ProductType → StorageFamily mapping. Caller doesn't know which rack — decoupled.

**Details:**
- Caller code: `StorageManager.Instance.AddToStorage(ProductType.Plank, 4)` → returns overflow (4 if no rack, 0 if all stored)
- StorageManager internally: ProductType → StorageFamily lookup (e.g., Plank → StorageFamily.Plank) → query StorageRackRegistry → call `rack.Add(productType, count)`
- Adding new product type = add to ProductType enum + add mapping → existing producers (machines) work without changes
- Mapping defined in StorageFamilyMapping (static class or SO)
- ProductType : Log, Stump, Firewood, WoodChips, Plank, Bark, Pellet, Fertilizer, Furniture
- StorageFamily: Log, Stump, Firewood, WoodChips, Plank, Bark, PelletBag, FertilizerBag, Furniture

**Why it matters:** Without router, every producer needs to know about every rack. With router, producers just say "I made a Plank" — system figures out where.

#### #122 ★ Dictionary<ProductType, int> warehouse rejestr (zero physical, auto-init)
- **Slug:** dictionary-warehouse-registry
- **Path:** engine/patterns/dictionary-warehouse-registry.md
- **Type:** A
- **Template:** pattern
- **Sources:** e55dbfd6-bb50, acb055d2-01aa
- **Related:** #121, #100
- **Tags:** unity, storage, dictionary, registry, data-driven

**Brief:** Warehouse inventory backed by `Dictionary<ProductType, int>` (key=type, value=count). Zero physical objects scattered in scene. New types auto-initialize to 0 on first access.

**Details:**
- `Dictionary<ProductType, int> inventory = new()` in StorageManager
- Access pattern: `inventory.TryGetValue(type, out var count)` → returns 0 if absent (auto-init)
- Add: `inventory[type] = (inventory.TryGetValue(type, out var c) ? c : 0) + amount`
- Visual layer reads dictionary, instantiates Visual prefabs per slot (e.g., 1 visual stack = 10 logs — see #100)
- ISaveable: serialize as JSON dict, restore on load
- Performance: O(1) lookup vs. iterating GameObjects
- Extension by data: new ProductType enum value = automatically tracked, zero code changes

**Why it matters:** "100 logs in storage" doesn't need 100 log GameObjects. Dictionary + visual abstraction = 60fps stable regardless of inventory size.

#### #123 Storage activation gating via upgrade purchase
- **Slug:** storage-activation-gating-upgrade
- **Path:** engine/patterns/storage-activation-gating-upgrade.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12
- **Related:** #096-119, #134
- **Tags:** unity, storage, upgrade, gating, progression

**Brief:** Storage racks start `isActive = false`. Unlocked progressively via `SawmillManager.OnUpgradePurchased` event (subscription in StorageRack.OnEnable).

**Details:**
- StorageRack.isActive default = false (rack exists in scene but rejects deposits)
- SawmillManager.OnUpgradePurchased(UnlockType unlockType, string unlockId) → fires when player buys upgrade
- StorageRack subscribes: `if (unlockType == UnlockType.StorageRack && unlockId == this.gameObject.name) isActive = true`
- Active on level 1: Log, Stump, Firewood_Regular (player needs these from day 1)
- Locked initially: Plank, Bark, Pellet, Fertilizer (require upgrade purchases)
- Visual: inactive rack rendered with placeholder material (transparent gray); active rack uses normal material
- Tutorial guides player to first upgrade ~5 minutes into demo

**Why it matters:** Progressive unlock keeps demo pacing — player doesn't drown in options on minute 1, gradually unlocks systems through play.

#### #125 CrateManager tier progression
- **Slug:** crate-manager-tier-progression
- **Path:** engine/patterns/crate-manager-tier-progression.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12
- **Related:** #123, #124
- **Tags:** unity, crate, tier, upgrade, inventory

**Brief:** CrateManager singleton tracks current crate tier (Small_10 → Medium_15 → Large_25 slots). Upgrade via SawmillManager.OnUpgradePurchased event subscription.

**Details:**
- Crate = player's car flatbed slots (loaded products waiting for delivery to customer)
- Tier enum: `CrateTier { Small_10, Medium_15, Large_25 }` (number = slot count)
- `List<CrateSlot> slots` where CrateSlot = (ProductType type, int amount)
- Methods: LoadItem(type, amount) → bool, UnloadAll() → list of slots, GetCurrentLoad() → int
- Upgrade event: `if (unlockType == UnlockType.Crate) UpgradeTier()`
- Visual: car flatbed shows different number of slot markers based on tier
- ISaveable: serialize tier + slot contents

**Why it matters:** Crate tier = direct measure of player progression. Each tier upgrade is a visible "promotion" moment.

#### #129 Architectural elements naming convention
- **Slug:** architectural-naming-convention
- **Path:** engine/patterns/architectural-naming-convention.md
- **Type:** A
- **Template:** pattern
- **Sources:** fda34b50-7e59, a45f1b8d-9d0b
- **Related:** #130, sweep2-1
- **Tags:** unity, blender, naming, architecture, prefabs

**Brief:** Strict naming convention for architectural elements: `Beam_{Direction}_{Position}_Log`, `Pillar_{Compass}_{Tall|Short}`, `Wall_{Compass}` (or `Wall_Triangle_{Compass}` for gables), `Foundation`.

**Details:**
- Beams: `Beam_North_Log`, `Beam_East_Top_Log`, `Beam_Center_Log` (suffix `_Log` confirms cylinder/log shape)
- Pillars: `Pillar_NE_Tall`, `Pillar_NW_Short`, `Pillar_SE_Tall2` (compass + height variant)
- Walls: `Wall_East`, `Wall_North`, `Wall_Triangle_North` (compass + gable indicator)
- Foundation: single `Foundation` (no variants typically)
- Roofs: `Roof` (single piece per building, or `Roof_North` / `Roof_South` for split roofs)
- Why: visual scanning of hierarchy in Unity Inspector + Blender Outliner is faster with predictable names
- Why: enables scripted setup (e.g., `Setup All Beams` script finds objects via prefix match)

**Why it matters:** "What's `cube.001.002`?" — bad. "What's `Beam_North_Log`?" — obvious. Naming is documentation.

#### #130 Collider distribution rule (BoxCollider per arch, beams no collider)
- **Slug:** collider-distribution-rule
- **Path:** engine/patterns/collider-distribution-rule.md
- **Type:** A
- **Template:** pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** #129
- **Tags:** unity, colliders, performance, architecture, level-design

**Brief:** Architectural collider distribution: BoxCollider per Foundation, Wall, Pillar (player can't pass through). Beams: NO collider (visual only, can pass under).

**Details:**
- Foundation: BoxCollider matching extents (player can walk on, can't fall through)
- Walls: BoxCollider per wall (player blocked from passing through)
- Pillars: BoxCollider per pillar (player can't clip through)
- Beams (cylindrical): NO collider — too high for player to interact with, collision adds cost for zero gameplay benefit
- Roofs: NO collider (player can't access roof in TT — would need ladder system)
- Why beams no collider: simpler player navigation (no head-bumping on doorways), fewer raycast hits, smaller physics scene

**Why it matters:** Adding colliders to everything = bloated physics scene. Selective collider use = smooth player movement + 60fps physics step.

#### #133 ★ Reputation backend migration with rollback safety
- **Slug:** reputation-backend-migration-rollback-safety
- **Path:** engine/patterns/migration-pattern-rollback-safety.md
- **Type:** D
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #037, #038
- **Tags:** unity, refactoring, migration, rollback, architecture

**Brief:** Backend migration pattern: split into 2 sprints (4a backend + 4b UI), independent commits, smoke test per stage. Migration = etapy z rollback safety, NEVER big-bang.

**Details:**
- Migration: `SawmillManager.CurrentReputation` → `ReputationManager` singleton (new dedicated manager)
- Sprint 4a: extract data + state, deprecate old API, smoke test sprzedaż NPC → rep rośnie → level reputacji = AvailableLevel
- Sprint 4b: UI consumers (HUD widget, kiosk) — non-architectural, low-risk
- Each sprint = independent git commit (easy rollback)
- Smoke test between sprints: verify old functionality still works
- API deprecation: leave old `SawmillManager.CurrentReputation` for now, mark `[Obsolete("Use ReputationManager.Instance.Current")]`
- Remove deprecated code only after grep confirms zero remaining callers (similar to #038 checklist)
- Communicate scope: 4a is "scary" (touches save format if not careful), 4b is "boring" (just UI)

**Why it matters:** Big-bang migrations break saves and cascading bugs. Etapowane migracje = each stage independently verifiable, minimum window for breakage.

#### #134 ReputationLevels.asset data-driven progression (5→13 levels)
- **Slug:** reputation-levels-data-driven
- **Path:** engine/patterns/reputation-levels-data-driven.md
- **Type:** A
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #133, #108
- **Tags:** unity, scriptableobject, progression, data-driven, reputation

**Brief:** Reputation levels defined in `ReputationLevels.asset` (SO), scaled from 5 to 13 levels by editing the asset. Code knows zero specific levels — reads from data.

**Details:**
- ReputationLevelsSO contains: `List<ReputationLevel>` where each entry has `threshold` (rep points required), `name` (e.g., "Nowicjusz", "Drwal", "Tartacznik"), `unlocksAtThisLevel`
- ReputationManager.GetCurrentLevel(): iterate list, return highest index where `threshold <= currentRep`
- Adding levels: edit asset in Inspector → add new entries → save. Zero code changes.
- Initial design: 5 levels (mvp scope). Scaled to 13 during balancing (more granular progression).
- Names localized via LocalizationManager (key = level name) — `Reputation.Level.Nowicjusz` etc.

**Why it matters:** Hard-coded levels = 13 code changes per balance pass. SO-driven = 13 Inspector clicks.

#### #135 KioskInteractable + Cube placeholder pattern
- **Slug:** kiosk-interactable-cube-placeholder
- **Path:** engine/patterns/kiosk-interactable-cube-placeholder.md
- **Type:** C
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #040, #133
- **Tags:** unity, interactable, kiosk, placeholder, ui

**Brief:** Diegetic kiosk (upgrade shop interaction point) = Cube primitive placed by designer + IInteractable component + `OnInteract → UpgradeShopUI.Instance.Open()`.

**Details:**
- Kiosk = world-space interactable, not UI overlay button (player walks up to kiosk to upgrade)
- Implementation: Cube primitive at SalesCounter position (or similar gameplay hub)
- Components: BoxCollider (raycast target) + KioskInteractable script (IInteractable)
- `OnInteract` callback: `UpgradeShopUI.Instance.Open()` (panel opens, blocks gameplay input via #117)
- Placeholder Cube can be swapped for proper model later (e.g., wooden notice board, computer terminal)
- Cleanup post-migration: remove `#if UNITY_EDITOR U key` shortcut once kiosk works (see #136)

**Why it matters:** Diegetic UI feels grounded. Cube as placeholder = designer can position immediately, art replaces later. Decoupled.

#### #004 ★ ANTI-PATTERN: Smart UV Project + Cycles bake for solid colors
- **Slug:** cycles-bake-for-solid-colors-antipattern
- **Path:** engine/anti-patterns/cycles-bake-for-solid-colors.md
- **Type:** A
- **Template:** anti-pattern
- **Sources:** (sweep2 — props pipeline)
- **Related:** #003, #001
- **Tags:** blender, baking, uv, anti-pattern, static-props

**Brief:** For static props with solid color regions (PelletBag, FirewoodBundle), don't use Smart UV Project + Cycles bake. Failure modes: black solid colors, blurred edges, ghost UV islands. Use numpy-based atlas + explicit UV instead.

**Details:**
- Symptom 1: solid color regions bake as BLACK (Cycles considers them un-lit surfaces, ambient occlusion overpowers)
- Symptom 2: detail edges blurred where UV islands meet (Cycles sampling averages neighbors)
- Symptom 3: ghost UV islands (Smart UV Project creates micro-islands invisible in viewport, baked output gets bleed)
- Why this fails: Cycles bake is designed for lit/shaded surfaces, not flat color atlases
- Correct approach for static props with solid colors (see #003):
  - numpy in Blender Python: create atlas PNG with explicit color blocks
  - Manual UV layout: per-face UVs to corresponding atlas region
  - Single material slot from start
- Reserve Cycles bake for materials with actual shading (procedural wood grain, complex shaders → bake to PNG normal/AO)

**Why it matters:** Wasted 3 hours on PelletBag debugging "why are my bags black." The technique itself is wrong for the use case — switch techniques, problem disappears.

#### #014 ★ ANTI-PATTERN: Scene files are BINARY, never edit as text
- **Slug:** scene-files-binary-never-text-edit
- **Path:** engine/anti-patterns/scene-files-binary-never-edit.md
- **Type:** D
- **Template:** anti-pattern
- **Sources:** (from 42-originals)
- **Related:** #013, #038
- **Tags:** unity, scene, yaml, prefab, missing-script

**Brief:** Unity scene/prefab files appear as YAML in text editors but are actually binary-ish with internal references (fileID, guid, m_LocalIdentfierInFile). Editing as text breaks scene. Recovery for missing scripts = manual Inspector "Remove Component."

**Details:**
- Scene `.unity` and prefab `.prefab` files: open as YAML text, but contain auto-generated GUIDs and file IDs
- Text-edit risks: change wrong line → scene fails to load → entire scene lost
- Specifically NEVER text-edit:
  - To "rename" a component (use Inspector)
  - To "move" GameObjects between scenes (use Hierarchy drag)
  - To "fix" missing scripts (use Inspector Remove Component)
- Acceptable text-edit cases: bulk grep for class references (`grep "MyClass" *.unity`), source-control diffs (read-only review)
- Recovery when scripts go missing (script file deleted but scene still references): scene loads with `m_Script: {fileID: 11500000, guid: 00000…}` errors → open scene → for each GameObject with missing script → Inspector → ⋮ menu → Remove Component
- Force text serialization for source-control diffability: Edit → Project Settings → Editor → Asset Serialization → Force Text

**Why it matters:** Text-edited scene that breaks = potentially unrecoverable (depends on backup state). Inspector edits = always undoable, always safe.

#### #042 ★ ANTI-PATTERN: Legacy code conflict after refactor
- **Slug:** legacy-code-conflict-after-refactor
- **Path:** engine/anti-patterns/legacy-code-conflict-after-refactor.md
- **Type:** A
- **Template:** anti-pattern
- **Sources:** (from 42-originals)
- **Related:** #037, #038
- **Tags:** unity, refactoring, legacy-code, conflict, technical-debt

**Brief:** After refactor moves functionality (e.g., `SpawnStump` moves from ChoppableTree to TreeFactory), old caller code still references old location. Two implementations diverge silently — bugs in production.

**Details:**
- Symptom: refactor seems clean, code builds, smoke test passes — but specific edge case still uses old path
- Example: `ChoppableTree.SpawnStump` was deprecated; new code uses `TreeFactory.CreateStump`. But one quest script still calls `ChoppableTree.SpawnStump` — works initially, diverges after first feature change to TreeFactory
- Why hard to detect: code looks correct in isolation, only fails when paths diverge
- Prevention:
  1. Delete old API entirely (Obsolete attribute or hard-remove) — forces callers to migrate
  2. If can't delete (compat reasons): old API delegates to new (`SpawnStump() => TreeFactory.CreateStump()`) — single source of truth
  3. Search before refactor: `grep -r "SpawnStump" Assets/` finds all callers
- This pattern bit us at least 3 times — codifying as anti-pattern

**Why it matters:** Refactor without cleanup = silent technical debt accumulation. Eventually production bug from drift. Better to break loud (Obsolete) than fail silent.

#### #054 ★ ANTI-PATTERN: Rotating Directional Light = black terrain at dawn/dusk
- **Slug:** rotating-directional-light-black-terrain
- **Path:** engine/anti-patterns/rotating-directional-light-day-night.md
- **Type:** A
- **Template:** anti-pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #055, #056
- **Tags:** unity, lighting, day-night, directional-light, ambient

**Brief:** First-pass day/night system rotates Directional Light to simulate sun position. At dawn/dusk, light shines horizontally → entire scene goes BLACK from steep angle. At night, "sun" shines from below ground → upside-down lighting.

**Details:**
- Naive approach: `directionalLight.transform.rotation = Quaternion.Euler(timeOfDay * 360, ...)` → sun "orbits" scene
- Failure mode 1: dawn/dusk angles produce extreme shadows (everything lit from horizon → mostly shadow)
- Failure mode 2: night angles point UP into sky (sun under map) → ambient becomes "shadow up" — surreal
- Failure mode 3: at exact horizon, NaN in shadow computation → flickering black artifacts
- Better approach (see #055): static overhead Directional Light + skybox handles "sun position" decoratively (Minecraft-style)
- Even better (see #056): 4-phase weighted color blending, ambient tracks time-of-day directly (not light rotation)
- Light intensity scales with time: midday 1.0, dawn/dusk 0.5, night 0.2 (but rotation stays static)

**Why it matters:** Wasted 2 days iterating on rotation-based day/night before realizing the approach is fundamentally wrong for the aesthetic we want. Codify so we don't repeat.

#### #112 ★ ANTI-PATTERN: Script overrides prefab Inspector values (CapsuleCollider case)
- **Slug:** script-overrides-prefab-inspector
- **Path:** engine/anti-patterns/script-overrides-prefab-inspector.md
- **Type:** D
- **Template:** anti-pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** sweep2-2, #113, #022
- **Tags:** unity, prefabs, inspector, scripts, debugging

**Brief:** ChoppableTree code added CapsuleCollider with hardcoded values (radius=0.0075, height=0.0024) → microscopic collider → trunk levitates / passes through ground. Fix: "use existing collider from prefab" pattern, NEVER overwrite Inspector-set values from code.

**Details:**
- Bug pattern:
  ```csharp
  CapsuleCollider col = trunk.AddComponent<CapsuleCollider>(); // duplicates if already exists
  col.radius = 0.0075f;  // microscopic — bug
  col.height = 0.0024f;  // microscopic — bug
  ```
- Result: collider too small to register collision with ground → Rigidbody.useGravity drops trunk through floor or stuck mid-air
- Symptom: tree falls, trunk spawns, levitates 2m above ground OR passes through terrain
- Fix pattern:
  ```csharp
  CapsuleCollider col = trunk.GetComponent<CapsuleCollider>();
  if (col == null) {
      col = trunk.AddComponent<CapsuleCollider>();
      col.direction = 0;  // X-axis for lying log (see #113)
      col.radius = 0.15f;
      col.height = 3.5f;
      col.center = Vector3.zero;
  }
  // If exists: DO NOT overwrite. Trust Inspector values.
  ```
- General rule (also covered in sweep2-2): if a prefab field is designer-tunable, code MUST NOT overwrite it
- Detection: if Inspector edit produces no in-game change, grep for the field name + `=` in code

**Why it matters:** Burned 90 minutes debugging "why does trunk fly into space." Anti-pattern codified — never debug it again.


---

### BATCH 5 — genre/tycoon/patterns (entries #020-#100, TT-specific)

#### #020 ★ NPC parking PD controller
- **Slug:** npc-parking-pd-controller
- **Path:** genre/tycoon/patterns/npc-parking-pd-controller.md
- **Type:** C
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #017, #024
- **Tags:** unity, npc, parking, pd-controller, vehicle, physics

**Brief:** NPC vehicle parks itself in slot via PD (Proportional-Derivative) controller: Kct=5 corrective torque, Kd=0.4 damping, invertSlotForward for back-in parking.

**Details:**
- NPCParkingPDController applies torque + force based on slot target position/rotation
- Proportional term: torque proportional to angular error from slot forward direction
- Derivative term: damping proportional to angular velocity (prevents oscillation)
- Constants: `Kct = 5` (correction gain), `Kd = 0.4` (damping gain) — tuned by trial
- `invertSlotForward = true` for back-in parking (NPC reverses into slot)
- Forward axis correction: `-transform.right` (see #017 for FBX quirk)
- Stop condition: position within 0.3m AND angular error within 5° AND linear velocity < 0.1m/s → switch to kinematic
- Works for both player parking lot and sawmill customer slots

**Why it matters:** Hand-coded "drive forward, then turn, then back up" logic = brittle. PD controller = robust, handles any starting position, looks natural.

#### #024 ★ Pipeline-style NPC spawn (OnPurchaseComplete trigger)
- **Slug:** pipeline-style-npc-spawn
- **Path:** genre/tycoon/patterns/pipeline-style-npc-spawn.md
- **Type:** C
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #025, #027
- **Tags:** unity, npc, spawn, pipeline, event-driven

**Brief:** NPC customer spawning triggered by purchase completion (not random/timed). When customer finishes purchase, next NPC spawns and drives to nearest free slot. Pipeline ensures continuous flow.

**Details:**
- Event: `SalesCounter.OnPurchaseComplete` raised when customer finishes transaction
- NPCSpawner subscribes: spawn next NPC at distant spawn point, set destination to free parking slot
- Free slot detection: iterate `ParkingSlotManager.slots`, find first `slot.occupant == null`
- If no free slots: queue NPC at distant location, spawn into slot when one opens
- Initial fill on load: spawn N NPCs at game start (no purchase event yet — see #025)
- Customers per minute: tunable via `SpawnRate` parameter (3-5 NPCs/min for active sawmill feel)
- Prevents: dead time (no customers in scene), overcrowding (too many at once)

**Why it matters:** Random/timed spawn = either empty parking lot or chaos. Pipeline spawn = steady customer flow scaled to player throughput.

#### #025 ★ Initial fill on load (don't serialize NPC state)
- **Slug:** initial-fill-on-load
- **Path:** genre/tycoon/patterns/initial-fill-on-load.md
- **Type:** A
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #024, #027, #068
- **Tags:** unity, npc, save-load, initial-state, lifecycle

**Brief:** NPC customers NOT serialized in save files. On scene load, NPCSpawner does an initial fill (spawn N customers into free slots). Persisted state would create save bloat + edge cases (mid-transaction NPC).

**Details:**
- ISaveable for: PARKING SLOT occupancy state (slot N has customer? yes/no, what they're buying)
- NOT ISaveable for: actual NPC GameObjects (mid-walk, mid-driving, mid-transaction)
- On load: read slot occupancy, spawn NPCs directly into slots, set their state to "shopping at slot X"
- Avoids: serializing 3D position/rotation of N NPCs, animation state, AI sub-state
- Initial fill count: scaled to current player progression (more customers if higher Reputation level)
- Edge case: if all slots empty at save, no initial fill needed (NPCs spawn naturally as gameplay resumes)

**Why it matters:** Save file size + save complexity stay manageable. Reload feels seamless — you see customers, but they aren't the same ones from before the save.

#### #026 NavMesh + kinematic waypoints hybrid
- **Slug:** navmesh-plus-kinematic-waypoints
- **Path:** genre/tycoon/patterns/navmesh-plus-kinematic-waypoints.md
- **Type:** C
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #020, #024
- **Tags:** unity, navmesh, npc, pedestrians, vehicles, hybrid

**Brief:** TT uses NavMesh for pedestrians (NPCs walking from car to counter) + kinematic waypoint sequence for vehicles (cars driving to/from parking). Different agents, different needs.

**Details:**
- Pedestrian NPCs: NavMeshAgent on standard NavMesh baked from terrain + walkable surfaces
- Vehicle NPCs: kinematic Rigidbody following pre-defined waypoint paths (entry road → parking lane → slot)
- Why split: NavMesh poor for vehicles (no concept of turning radius, can't reverse), perfect for free-form walking
- Vehicles "arrive" at slot via waypoint, then PD controller handles final park (see #020)
- Walkable NavMesh excludes road surface (NPCs don't walk in road)
- Vehicle path baked separately as `List<Transform> waypoints` per spawn point

**Why it matters:** Treating cars like pedestrians produces unrealistic driving. Hybrid approach = each agent type uses its appropriate movement system.

#### #027 Object pooling for NPCs + FIFO queue
- **Slug:** object-pooling-npcs-fifo-queue
- **Path:** genre/tycoon/patterns/object-pooling-npcs-fifo-queue.md
- **Type:** A
- **Template:** pattern
- **Sources:** 907b0de5-3c4c
- **Related:** #024, #075
- **Tags:** unity, npc, object-pool, performance, fifo

**Brief:** Maximum 8 active NPCs at a time, returned to object pool when done. SalesCounter has FIFO queue of waiting customers — pool reused as customers cycle through.

**Details:**
- Pool: 8 inactive NPC GameObjects pre-instantiated at scene load
- Spawn: pull from pool, position at spawn point, activate
- Despawn: deactivate, reset state, return to pool
- FIFO queue: `List<NPCCustomer> waitingQueue` at SalesCounter, customer joins back of queue when arriving at counter
- Counter serves head of queue, after sale completes → customer despawns, queue shifts forward
- Performance: avoid GC spikes from Instantiate/Destroy
- Future-proof: tier-based parking (2/4/8/12 slots) → pool size scales with progression

**Why it matters:** Allocate/free 100 NPCs/hour = GC churn. Pool 8 NPCs = predictable memory, zero spikes.

#### #043 ★ Quantity-not-quality design principle
- **Slug:** quantity-not-quality-design-principle
- **Path:** genre/tycoon/decisions/quantity-not-quality-principle.md
- **Type:** C
- **Template:** decision
- **Sources:** 03ec0ede-bc12, dab6f299-7ec0
- **Related:** #072, #044
- **Tags:** game-design, tycoon, minigame, quality, quantity, progression

**Brief:** TT minigames affect quantity, not quality. Tree species = separate SKU (different price, not a quality grade). Machine tier = gate on tree tiers (T1/T2/T3), NOT a quality cap. See tier-system-foundation.md. Minigame result (bad/good/perfect) = quantity multiplier (1/2/3 outputs).

**Details:**
- Design decision: separate "skill" from "content"
- Quality determined by inputs (tree species + machine tier), NOT player skill
- Quantity determined by minigame execution (skill)
- Why: player feels growth via skill (more outputs per cycle) without "missing out" on premium content (Spruce + bad result still gives 1 Plank, just fewer)
- Alternative considered (rejected): minigame determines QUALITY (bad=Plank_Regular, perfect=Plank_Premium) — would punish casual players too hard
- NPC workers always produce "Good" tier (consistent 2x output) regardless of player skill — sets ceiling for automation
- Late-game upgrade: `perfectQuality = true` on worker = always Perfect (3x output), endgame progression goal

**Why it matters:** Foundational design decision. Affects all minigames + worker design + balance. Documented so future features respect it.

#### #046 ★ Carry Capacity progression (1→3→5→8) + sprint advantage
- **Slug:** carry-capacity-progression-sprint
- **Path:** genre/tycoon/patterns/carry-capacity-progression-sprint.md
- **Type:** C
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #043, #073
- **Tags:** game-design, tycoon, progression, carry, sprint, npc

**Brief:** Player carry capacity progresses 1 → 3 → 5 → 8 logs via upgrades (Rękawice robocze, Szelki drwala, Siła weterana). Combined with sprint, player always outperforms NPC worker (capped at 3).

**Details:**
- Level 0 (start): 1 log in hands
- Level 1 (Rękawice robocze upgrade): 3 logs (wiązka pod pachą)
- Level 2 (Szelki drwala upgrade): 5 logs (stos na ramieniu)
- Level 3 (Siła weterana upgrade): 8 logs (duży stos)
- Visual: arm pose changes per level, log count visible in hands
- NPC worker: always 3 logs, walking speed only (no sprint), 200ms delay at storage rack ("searching" time)
- Player advantage = sprint (Shift = 2x speed) + higher capacity + zero delay = always faster despite Level 0 disadvantage if walking
- Late game: player at Level 3 + sprint = 5x throughput vs NPC, justifies player presence in production loop

**Why it matters:** Tycoon must reward player presence. If NPC equals player, players AFK. Capacity + sprint asymmetry = player always faster.

#### #048 Multi-step quest checklist pattern
- **Slug:** multi-step-quest-checklist
- **Path:** genre/tycoon/patterns/multi-step-quest-checklist.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12, fda34b50-7e59
- **Related:** #047, #049
- **Tags:** unity, quest, ui, tutorial, multi-step

**Brief:** Quests with multiple sub-tasks display as in-line checklist (Quest 5: "Zbierz 5 kłód → [✓] [✓] [ ] [ ] [ ]"). Updates in real-time as sub-tasks complete.

**Details:**
- QuestSO defines: `List<QuestStep>` where each step has description + completion condition + counter (for "collect N" style)
- QuestUI renders checkbox per step (✓ complete, ☐ incomplete) + counter
- Real-time updates via QuestManager.OnStepProgress event
- Steps can be sequential (each unlocks next) or parallel (any order)
- Example quests in TT:
  - Quest 1 (CleanUp): collect 6 debris → repair counter — sequential
  - Quest 2 (GetAxe): pick up axe → no steps, single condition
  - Quest 5 (Wycinka): chop 5 trees → spawn 5 saplings — sequential per tree
- 9 quests total in MVP demo

**Why it matters:** "Complete X" without progress = player guesses how close they are. Checklist = explicit progress, tangible.

#### #049 Customer tier system (Regular/Contractor/VIP)
- **Slug:** customer-tier-system
- **Path:** genre/tycoon/patterns/customer-tier-system.md
- **Type:** C
- **Template:** pattern
- **Sources:** fda34b50-7e59
- **Related:** #024, #043
- **Tags:** game-design, tycoon, customer, tier, progression

**Brief:** NPC customers have 3 tiers: Regular (small orders, ~$50-100), Contractor (medium, ~$300-500), VIP (large, $1000+). Tier unlocks based on player Reputation level — early game = Regular only.

**Details:**
- Regular: 5-10 of a product, common species, low margin
- Contractor: 15-30 of a product, often premium species, medium margin
- VIP: 50+ of a product, specific premium species, high margin, often time-limited
- Spawn distribution per Reputation level:
  - Lvl 1-2: 100% Regular
  - Lvl 3-5: 70% Regular, 30% Contractor
  - Lvl 6-8: 50% Regular, 40% Contractor, 10% VIP
  - Lvl 9+: 30% Regular, 50% Contractor, 20% VIP
- Visual differentiation: different car models per tier (Regular: economy car, VIP: luxury sedan)
- Demo Day 4 "FINAL PUSH": first VIP customer arrives ($1500 mansion order) — climactic milestone

**Why it matters:** Progression feels meaningful. "I just sold to a VIP" = memorable moment. Player works toward unlocking better customers.

#### #053 WaterZone gameplay component
- **Slug:** water-zone-gameplay-component
- **Path:** genre/tycoon/patterns/water-zone-gameplay-component.md
- **Type:** C
- **Template:** pattern
- **Sources:** 6967a837-dc86
- **Related:** #052, #051
- **Tags:** unity, water, wading, slowdown, gameplay-zone

**Brief:** WaterZone component on river/cave water surface: BoxCollider (trigger) + speedMultiplier (0.5 for wading) + wadingDepthPercent (clamp player Y to water surface - depth).

**Details:**
- Trigger collider over water area
- Player enters trigger → `PlayerController.speedMultiplier *= 0.5f` (50% movement)
- Wading depth: clamp player Y to `waterSurface.y - 0.6f` (player submerged to waist visually)
- Audio: splash on enter, wade loop while inside, splash on exit
- Player exit: restore speedMultiplier to 1.0
- For "stuck in water" or "deep water = death" considered (rejected for MVP — keep gameplay forgiving)
- Visual: water shader unaffected by player presence (no ripples — would need GPU work)

**Why it matters:** Player can cross river instead of going around bridge → strategic choice. Slower but shorter route. Adds map richness without complex swimming system.

#### #055 ★ Minecraft-style lighting: static overhead + decorative sun/moon
- **Slug:** minecraft-style-lighting
- **Path:** genre/tycoon/patterns/minecraft-style-lighting.md
- **Type:** C
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50
- **Related:** #054, #056, #057
- **Tags:** unity, lighting, day-night, minecraft-style, low-poly

**Brief:** Static overhead Directional Light (always pointing down) + decorative sun/moon in skybox shader. Light intensity varies with time of day, but ROTATION never changes. Eliminates anti-pattern #054.

**Details:**
- Directional Light setup: rotation locked to (~45°, 0°, 0°) — overhead-ish, never moves
- Intensity scales with `TimeOfDay`: midday 1.0, dawn/dusk 0.5, night 0.2 (but never 0 — minimum 55% per design rule)
- Color also scales: midday white, dawn warm orange, dusk red-orange, night cool blue
- Skybox handles "sun position" decoratively (see #057) — visually sun rises/sets, but no actual light source moves
- Result: no horizontal-angle issues (see #054), consistent shadowing throughout day, "stylized" feel
- Inspired by Minecraft, Astroneer, similar low-poly games where realism subordinate to playability

**Why it matters:** Realism for lighting = bugs (see #054). Stylized = stable, predictable, every angle plays well.

#### #061 ToolWheel UX pattern
- **Slug:** tool-wheel-ux-pattern
- **Path:** genre/tycoon/patterns/tool-wheel-ux-pattern.md
- **Type:** C
- **Template:** pattern
- **Sources:** fd9cc5b6-bb50, a45f1b8d-9d0b
- **Related:** #062, #117
- **Tags:** unity, ui, tool-wheel, fps, controls

**Brief:** Tool selection via radial wheel: hold Q to open, mouse selects sector, LPM confirms / release Q cancels. Mouse look disabled during wheel. No game pause (real-time pressure).

**Details:**
- Hold Q: ToolWheel UI opens (radial menu, sectors = available tools), mouse cursor unlocks
- Mouse selects sector: highlight on hover, no commitment yet
- LPM: confirm selection, close wheel, equip tool
- Release Q without LPM: cancel, close wheel, no change
- During wheel: PlayerController.canMove = false (see #117) BUT game NOT paused (Time.timeScale = 1)
- Available tools depend on progression: Axe (Quest 2), Shovel (after first stump), Planting (after first sapling)
- Locked tools shown grayed out with unlock hint
- Inspired by GTA V / Borderlands tool wheels

**Why it matters:** Inventory menu pauses game = breaks tension. Wheel doesn't pause = decisions feel real-time. UX feels modern.

#### #070 ★ WorkerData (SO blueprint) + WorkerInstance (runtime) split
- **Slug:** worker-data-instance-split
- **Path:** genre/tycoon/patterns/worker-data-instance-split.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12, a0ce5046-a69a
- **Related:** #071, #072, #108
- **Tags:** unity, scriptableobject, worker, blueprint, runtime

**Brief:** WorkerData (ScriptableObject blueprint): role, baseWorkSpeed, baseSalary, displayName. WorkerInstance (runtime class, NOT MonoBehaviour): isHired, workSpeedMultiplier, perfectQuality flag. Upgrades modify instance, not blueprint.

**Details:**
- WorkerData SO (designer-tuned):
  - `WorkerRole role` (SalesCounter, Chipper, PlankMaker, Pelletizer, FertilizerMaker, FurnitureWorkshop)
  - `float baseWorkSpeed` (seconds per cycle)
  - `int baseSalary` (cost per day)
  - `string displayName` (UI display)
- WorkerInstance (runtime, serialized in save):
  - `WorkerData blueprint` (reference to SO)
  - `bool isHired`
  - `float workSpeedMultiplier` (default 1.0, increased by upgrades)
  - `bool perfectQuality` (default false, true after lvl 13 upgrade)
- Stacking upgrades: workSpeedMultiplier *= 1.2 per "Speed +20%" upgrade (compound)
- Why split: blueprint = designer-tuned data, instance = player-progression state. Decoupled.

**Why it matters:** Upgrading a worker shouldn't mutate the SO (which is shared across all instances of that worker type). Instance state = clean separation.

#### #071 Worker design: SimulateWorkCycle, no NavMesh/AI
- **Slug:** worker-simulate-work-cycle
- **Path:** genre/tycoon/patterns/worker-simulate-work-cycle.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12
- **Related:** #070, #073
- **Tags:** game-design, tycoon, npc-worker, simulation, abstraction

**Brief:** NPC workers don't walk to machines or pick up logs visually — they "occupy" the workstation slot and `SimulateWorkCycle()` runs on timer. Pure logic, no pathfinding, no animation overhead.

**Details:**
- Worker assignment: player drags worker onto workstation in UI → worker "operates" that station
- SimulateWorkCycle (called every `baseWorkSpeed * workSpeedMultiplier` seconds):
  - Check raw materials available in storage
  - If yes: consume materials, produce outputs (added to storage via #121)
  - If no: idle (visual: worker idle animation at workstation)
- Visual: static worker model at workstation, occasional idle animation (no pathfinding visible)
- Trade-off: less immersive (vs Stardew Valley NPCs that walk around) but enables many workers without performance hit
- Late game design accommodation: 12+ workers across stations, all simulated abstractly

**Why it matters:** Visual fidelity vs scale trade-off. TT chose scale. 12 workers as logic-only = same performance as 1 worker. 12 workers walking around = framerate dies.

#### #072 Worker output quality (perfectQuality flag + random distribution)
- **Slug:** worker-output-quality-distribution
- **Path:** genre/tycoon/patterns/worker-output-quality-distribution.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12
- **Related:** #070, #043
- **Tags:** game-design, tycoon, worker, quality, distribution, progression

**Brief:** Worker output by default: 20% Bad / 50% Good / 30% Perfect (matches average player skill). Perfectquality flag = always Perfect (3x output, late game upgrade).

**Details:**
- Default GetOutputQuality(): random roll
  - 20%: Bad (1 output)
  - 50%: Good (2 outputs)
  - 30%: Perfect (3 outputs)
- perfectQuality = true: skip roll, always return Perfect (3 outputs)
- Achievable only at Sawmill Level 13 upgrade ("Maestro Workforce") — endgame goal
- Player can outperform default worker (60% Perfect rate with skill) → automation is "good enough" not "best"
- Maxed worker = parity with skilled player (3 outputs each), frees player for management tasks
- Distribution numbers tunable via SO (BalanceConfig.workerQualityDistribution)

**Why it matters:** Worker design must NEVER feel pointless OR overpowered. Default = "slightly worse than skilled player." Upgraded = "as good as skilled player." Sweet spot.

#### #073 OrderFulfiller interface (player vs NPC, player always faster)
- **Slug:** order-fulfiller-interface
- **Path:** genre/tycoon/patterns/order-fulfiller-interface.md
- **Type:** C
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #046, #072
- **Tags:** unity, interface, order-fulfillment, player, npc, asymmetry

**Brief:** Both player and NPC implement `IOrderFulfiller` interface (PickupProduct, DeliverToCounter, GetCarryCapacity). Identical contract, asymmetric implementation — player always faster (sprint, higher capacity, zero delay).

**Details:**
- Interface:
  ```csharp
  public interface IOrderFulfiller {
      int CarryCapacity { get; }       // 1/3/5/8 player, 3 NPC
      float MovementSpeed { get; }     // 5/10 (walk/sprint) player, 3 NPC
      float StorageDelay { get; }      // 0 player, 0.2s NPC
      IEnumerator FulfillOrder(NPCCustomer customer);
  }
  ```
- Player implementation: free movement, sprint, no delay
- NPC worker implementation: walking only, fixed delay at each rack ("searching")
- Both can fulfill same customer order — race for who gets there first
- Player presence = higher throughput, but NPC = throughput while AFK
- Late game: hire NPC to handle 50% of orders, focus player on premium customers

**Why it matters:** Asymmetric implementation behind same interface = both work, but player choice meaningful. Pattern reusable for any "manual vs automated" gameplay split.

#### #084 Wing snap-points modular instant fade-in
- **Slug:** wing-snap-points-modular-fade-in
- **Path:** genre/tycoon/patterns/wing-snap-points-modular-fade-in.md
- **Type:** C
- **Template:** pattern
- **Sources:** fda34b50-7e59
- **Related:** #083, #085
- **Tags:** unity, building, modular, snap-points, fade, expansion

**Brief:** Building expansion (sawmill: Strefa 1 → +Strefa 2 → +Strefa 3) uses pre-placed wing prefabs hidden initially, revealed with instant material fade-in on purchase. No physics simulation, no animation.

**Details:**
- All wings placed in scene during level design (Strefa 1 visible at start, Strefa 2 + 3 hidden)
- Hidden state: GameObject inactive OR material alpha 0
- On purchase: enable GameObject + fade material alpha 0 → 1 over 1.5s (lerp)
- "Snap points" = pre-placed wing prefabs, no runtime instantiation/positioning
- Visual VFX: dust burst at wing edge during fade (sells "construction" feeling)
- Audio: construction sound + ambient swell during reveal
- Inspired by Tavern Manager, Schedule I — instant satisfaction over realistic build animation
- Decision rationale (see #083): instant spawn = progression feels rewarded; realistic construction (Banished, Frostpunk) = progression feels slow

**Why it matters:** Tycoon progression must feel rewarding. Instant fade-in = "I bought it, I got it." Build animation = "I bought it, now wait 5 minutes." Worse UX.

#### #085 Debris cleanup = single-click drop materials visual
- **Slug:** debris-cleanup-single-click-drop
- **Path:** genre/tycoon/patterns/debris-cleanup-single-click-drop.md
- **Type:** C
- **Template:** pattern
- **Sources:** 1697ca97-c1ad
- **Related:** #084
- **Tags:** unity, tutorial, debris, cleanup, visual-progression

**Brief:** Cleanup debris (6 piles in starting sawmill) interaction: single E click = pile disappears, 1 wooden plank drops to ground. No minigame, no hold-interact, no animation. Visual feedback through dropped material.

**Details:**
- 6 DebrisCollectible GameObjects scattered in starting sawmill
- Each has BoxCollider + Interactable + DebrisCollectible script
- E interaction: pile despawns, instantiate 1 plank prefab at pile position (visible drop)
- Quest tracker updates: 6/6 → quest complete
- Material reward (the plank) used to repair Counter_Broken (next quest step)
- Why no minigame: tutorial moment, want player to feel agency NOT frustration
- Quest highlight (#047) draws player attention to next pile

**Why it matters:** First gameplay moment must be SIMPLE. Single click = "I did a thing!" Multi-step minigame on first interaction = "I'm confused."

#### #096+#119 ★ StorageRack family system [MERGED]
- **Slug:** storage-rack-family-system
- **Path:** genre/tycoon/patterns/storage-rack-family-system.md
- **Type:** C
- **Template:** pattern
- **Sources:** 03ec0ede-bc12, acb055d2-01aa
- **Related:** #120, #121, #122, #098
- **Tags:** unity, storage, family, capacity, racks, sawmill
- **Dedup:** Merger of R4 #96 (initial design) + R5 #119 (expanded with maxDistinctStacks + activation gating)

**Brief:** Each StorageRack has: `family` (StorageFamily enum: Log/Plank/Bark/Firewood/PelletBag/etc), `currentAmount`, `maxCapacity`, `isActive`, `maxDistinctStacks` (species variety cap). Auto-registers in StorageRackRegistry via OnEnable.

**Details:**
- StorageFamily enum: Log, Plank, Bark, Firewood, WoodChips, PelletBag, FertilizerBag, Furniture, Stump
- StorageRack component fields:
  - `StorageFamily family` (designer-set in Inspector)
  - `int maxCapacity` (total items, e.g., 10 Planks)
  - `int maxDistinctStacks` (species variety cap, e.g., 3 = max 3 species in this rack)
  - `bool isActive` (gating, see #123)
  - `int currentAmount` (runtime, derived from fillLevel children)
- maxDistinctStacks values by family:
  - Bark: 1 (only WoodSpecies.None)
  - Plank: 3 (3 species variety realistic for a single rack)
  - Log: 4 (handles 4 tree species)
  - PelletBag/FertilizerBag: 2 (typically same recipe across species)
- Auto-rejestracja in OnEnable (see #120)
- Visual fill alignment in fillLevel transforms (see #098)
- Reading: `StorageRackRegistry.TryGetRackByFamily(family, out rack)` → `rack.Add(productType, count)`

**Why it matters:** Storage architecture is data-driven. Adding a new family (e.g., "Toy") = add enum value, set racks, zero code. Foundational for content scaling.

#### #100 Visualization ratio (10:1 inventory to visual stack)
- **Slug:** visualization-ratio-inventory-to-visual
- **Path:** genre/tycoon/patterns/visualization-ratio.md
- **Type:** C
- **Template:** pattern
- **Sources:** e55dbfd6-bb50
- **Related:** #122, #098
- **Tags:** unity, visualization, inventory, ratio, scale

**Brief:** Storage visualization uses ratio: 10 units in inventory = 1 visual stack model. Predefined slot transforms per material type. Visual reads dictionary, instantiates N stack prefabs.

**Details:**
- Ratios per material type (in StorageFamilySO):
  - Logs: 10:1 (10 logs in inventory = 1 visual stack of 5-6 logs)
  - Planks: 20:1 (1 pallet of 20 planks)
  - Pellet bags: 5:1 (1 visual stack of 4-5 bags)
  - Bark/Sawdust: 50:1 (loose pile, abstract)
- Visual slot transforms placed in scene (or runtime-positioned in rack)
- Update logic:
  ```csharp
  int visualStacks = Mathf.FloorToInt(currentAmount / ratio);
  for (int i = 0; i < maxSlots; i++) slotVisuals[i].SetActive(i < visualStacks);
  ```
- Doesn't need to be precise (player won't count) — "looks full" is the goal

**Why it matters:** "1000 logs" needs to look distinct from "100 logs" without rendering 1000 log meshes. Ratio + slot system = visual fidelity at scale, performance preserved.

#### #132 Top-down camera minigame (stump digging — 4th mechanic)
- **Slug:** top-down-camera-minigame-stump-digging
- **Path:** genre/tycoon/patterns/top-down-minigame-stump-digging.md
- **Type:** C
- **Template:** pattern
- **Sources:** a45f1b8d-9d0b
- **Related:** #043, #044
- **Tags:** unity, minigame, camera, top-down, stump, mechanic

**Brief:** Stump digging minigame uses top-down camera + click-on-targets mechanic. 4th distinct minigame mechanic in TT alongside zone-click (tree chop), needle (sawmill), hold-with-needle (chipper).

**Details:**
- Camera position: directly above stump, looking down (10m elevation, orthographic-ish)
- 4-8 golden points appear around stump perimeter (Vector3 around stump.position)
- Player clicks each in sequence, each click = "dig" animation + spark VFX
- After all clicks: stump tilts + lifts out of ground (Rigidbody enabled, brief fall)
- Player collects stump as item (CollectableStump prefab)
- Camera transitions back to player FPP after collection
- Minigame catalog (4 distinct mechanics in TT):
  1. Zone-click (tree chop — strafe & click at correct moment)
  2. Circular cursor needle (sawmill — needle in spinning wheel)
  3. Hold-needle (chipper — hold E until needle in zone)
  4. Top-down click-targets (stump digging)
- Future: mouse drag tempo (#044, PlankMaker)

**Why it matters:** Variety prevents fatigue. 5 distinct mechanics in MVP = 5 different "moments." Game feels rich.


---

### BATCH 6 — genre/tycoon/decisions + survival + cross-genre (entries #050-#140)

#### #050 Sales flow decision (4 options A/B/C/D, chose D hybrid)
- **Slug:** sales-flow-decision-hybrid-d
- **Path:** genre/tycoon/decisions/sales-flow-decision-hybrid.md
- **Type:** C
- **Template:** decision
- **Sources:** 03ec0ede-bc12
- **Related:** #049, #073
- **Tags:** game-design, sales, customer-flow, decision-record

**Brief:** Sales flow design considered 4 options: A (player-only), B (auto-NPC), C (mixed manual+auto), D (hybrid — player + NPC worker behind counter, side-by-side throughput). Chose D for engagement + scalability.

**Details:**
- Option A — Player serves all customers: high engagement, low scalability (player overwhelmed late game). REJECTED.
- Option B — NPC worker handles everything: low engagement, full scalability. REJECTED (player AFK).
- Option C — Mixed manual + automatic: player chooses to manual or auto per order. REJECTED (decision fatigue).
- Option D — Hybrid: BOTH player and NPC worker active simultaneously, both serve customer queue. SELECTED.
  - Player advantages: sprint, higher capacity, no delay (see #073)
  - NPC advantages: 24/7 availability, frees player attention
  - Late game: hire 2nd NPC worker → 3x throughput
- Throughput scaling: 1 player ~ 4 customers/min, +1 NPC = +2 customers/min, +2 NPC = +4 customers/min
- Counter has 2 service slots (Counter_1, Counter_2) — player + NPC can serve different customers simultaneously

**Why it matters:** Foundational decision affecting all subsequent worker design. Codified to prevent regression to A/B/C in future planning.

#### #062 ★ Tool tier replacement (NOT multiple tiers in inventory)
- **Slug:** tool-tier-replacement-decision
- **Path:** genre/tycoon/decisions/tool-tier-replacement-not-inventory.md
- **Type:** C
- **Template:** decision
- **Sources:** fd9cc5b6-bb50, a45f1b8d-9d0b
- **Related:** #061, #043
- **Tags:** game-design, tool, tier, upgrade, inventory

**Brief:** Upgrading a tool (Axe T1 → T2 → T3) REPLACES existing tool. Player doesn't carry multiple tiers. Same applies to Shovel, future tools.

**Details:**
- Considered: keeping T1 + T2 + T3 in inventory (Stardew Valley approach) — REJECTED
- Reasoning: inventory bloat, decision fatigue ("which axe should I use for this tree?"), unclear upgrade benefit
- Decision: T2 PERMANENTLY replaces T1, T3 replaces T2. Old tier disappears.
- Visual: tool model changes when tier purchased, on toolwheel and viewmodel
- Upgrade cost = irrevocable (commitment to new tier)
- Why: tycoon progression should feel like always moving forward, no "old gear taking up space"
- Implication for save/load: only current tier saved, not history (T3 player can't downgrade to T1)

**Why it matters:** "Should I keep my old axe just in case?" — bad design. Forces players to think about gear management instead of tycoon strategy. Replacement = forward momentum.

#### #074 Decision: VFX wycofane (sawdust/kurz/liście)
- **Slug:** vfx-wycofane-decision
- **Path:** genre/tycoon/decisions/vfx-wycofane-decision.md
- **Type:** C
- **Template:** decision
- **Sources:** a0ce5046-a69a, dab6f299-7ec0
- **Related:** #075, #076
- **Tags:** game-design, vfx, scope, deferred, decision-record

**Brief:** Planned VFX (sawdust on tree cut, kurz on tree fall, sezonowe liście) cut from MVP. Reason: VFX budget (#075) limited, gameplay value low vs cost. Reconsidered post-launch.

**Details:**
- Originally planned: sawdust VFX on chop, kurz cloud on tree fall, liście fall in autumn season
- Cut from MVP because:
  - VFX system not core to "tycoon" experience (vs. immersive sim)
  - Each VFX requires: prefab design, particle config, perf tuning, audio sync
  - Time better spent on UI polish, customer variety, balance
- Retained VFX: vehicle exhaust (active road feel), sawmill smoke (active sawmill feel), splash (water interaction), gold coin sale (positive feedback)
- Decision: revisit after demo launch based on community feedback ("game feels lifeless" = add back)
- Lesson for future projects: itemize ALL planned VFX, score per gameplay-value-vs-cost, cut bottom 50% from MVP

**Why it matters:** Scope creep killer. Codify scope cuts so they don't quietly re-appear in feature lists.

#### #083 ★ Building progression: instant spawn post-purchase (Tavern Manager style)
- **Slug:** building-progression-instant-spawn
- **Path:** genre/tycoon/decisions/building-progression-instant-spawn.md
- **Type:** C
- **Template:** decision
- **Sources:** fda34b50-7e59
- **Related:** #084, #086
- **Tags:** game-design, building, progression, instant, tavern-manager

**Brief:** Building expansions appear INSTANTLY after purchase (Tavern Manager style), not gradually constructed. Inspired by feel of "I bought it, it's there." Realism-first construction (Banished, Frostpunk) rejected.

**Details:**
- Considered:
  - Realistic construction: NPCs build over 5-10 in-game minutes (Banished/Frostpunk style) — REJECTED
  - Instant spawn (T-shaped fade-in via #084) — SELECTED
- Reasoning:
  - Tycoon games reward player decisions immediately
  - "Wait 5 minutes for building" interrupts gameplay loop
  - Players will save-scum to bypass anyway
  - Realism: not core to TT aesthetic (low-poly, stylized)
- Wing expansion order: Strefa 1 (start) → Strefa 2 (right wing, mid-game) → Strefa 3 (left wing, late game)
- Each wing reveal: 1.5s fade-in + dust VFX + construction sound (sells "construction" feeling without time cost)

**Why it matters:** Pacing decision. Instant = "I'm in flow." Gradual = "I'm waiting." Flow > realism.

#### #086 ★ Player-built vs purchased dichotomy
- **Slug:** player-built-vs-purchased-dichotomy
- **Path:** genre/tycoon/decisions/player-built-vs-purchased-dichotomy.md
- **Type:** C
- **Template:** decision
- **Sources:** fda34b50-7e59, 1697ca97-c1ad
- **Related:** #085, #083
- **Tags:** game-design, building, player-agency, scope

**Brief:** Only ONE thing is "built" by player (Counter_Broken → Counter_Fixed). Everything else (wings, machines, kiosks) is purchased and instantly appears. Limits scope, focuses meaning.

**Details:**
- Player-built (manual, gives sense of agency):
  - Counter_Broken: collect 1 plank from cleanup, hold E to repair → Counter_Fixed appears
  - That's it. One tutorial moment of "I built this."
- Purchased + instant (everything else):
  - Sawmill wings (#083)
  - Machines (Chipper, PlankMaker, Pelletizer)
  - Kiosks
  - Workers
  - Vehicle upgrades
- Why: player-built mechanic costs designer time per building (custom flow + animation). Limited to 1 = scope manageable + tutorial moment.
- Future games can scale player-build (Eskimo Simulator: build igloo manually, tents, dog houses)

**Why it matters:** "Player builds everything" is scope nightmare. "Player builds one thing as tutorial" = best of both worlds.

#### #087 UI Phase 4 catalog (12 systems roadmap)
- **Slug:** ui-phase-4-catalog
- **Path:** genre/tycoon/decisions/ui-phase-4-catalog.md
- **Type:** C
- **Template:** decision
- **Sources:** 6967a837-dc86, a0ce5046-a69a
- **Related:** #093+94+95, #047
- **Tags:** game-design, ui, roadmap, scope, phase-4

**Brief:** UI Phase 4 catalog (12 systems planned for post-MVP UX completion): HUD complete, Notification Queue, Tooltip System, Cursor Management, Confirmation Dialogs, Visual Feedback, Typography, Icons, Statistics, Settings, Accessibility, Credits.

**Details:**
- All 12 systems planned, no implementations beyond placeholders:
  - HUD kompletny (money + day/time)
  - Notification Queue (info/warning/achievement toasts)
  - Tooltip System (inline help on UI hover)
  - Cursor Management (dynamic crosshair states)
  - Confirmation Dialogs (Tak/Nie modals)
  - Visual Feedback (floating text +$ / -$, progress bars over machines)
  - Typography / Fonts (#093+94+95)
  - Ikony UI (render from Blender, baked from 3D)
  - Statistics / Journal (#118)
  - Ustawienia gracza (audio/graphics/controls)
  - Accessibility (font scale, high contrast — see #093+94+95)
  - Credits Screen
- Roadmap: complete sequentially in Phase 4 (post-MVP, pre-EarlyAccess)
- Estimated effort: 2-3 weeks per system on average (UI work is slow)

**Why it matters:** Documenting Phase 4 scope prevents surprise late-game scope creep ("we forgot we need a settings menu!"). All known, all scoped.

#### #099 Rack architecture decision (3 options analyzed)
- **Slug:** rack-architecture-decision
- **Path:** genre/tycoon/decisions/rack-architecture-decision.md
- **Type:** C
- **Template:** decision
- **Sources:** dab6f299-7ec0
- **Related:** #096+119, #121
- **Tags:** game-design, storage, architecture, decision-record

**Brief:** Storage racks: 3 options considered. (A) Shared BagRack with capacity upgrades. (B) Dedicated rack per product. (C) Visual-only swap. Decision pending Creative Director.

**Details:**
- Option A: shared BagRack capacity upgrade
  - Pro: simple model, gracz odczuwa korzyść (more slots)
  - Con: one rack mixing all bag types (pellet + nawóz + meble) = visual chaos late game
- Option B: dedicated rack per produkt (PelletRack, FertilizerRack, etc.)
  - Pro: visual clarity, each rack visually distinct
  - Con: more racks = more space required, late game cluttered
- Option C: visual upgrade only (model swap)
  - Pro: zero gameplay change, pure cosmetic
  - Con: minimal player feedback, "I bought an upgrade but nothing changed"
- Decision: open. Hunter as Creative Director.
- Recommendation (Claude): Option B (dedicated) — best for visual clarity and "showroom" feel of sawmill

**Why it matters:** Document the decision-making process, not just the outcome. Future similar decisions can reference this reasoning.

#### #110 PlantingSpot universal-not-typed design
- **Slug:** planting-spot-universal-not-typed
- **Path:** genre/tycoon/decisions/planting-spot-universal-not-typed.md
- **Type:** C
- **Template:** decision
- **Sources:** e55dbfd6-bb50
- **Related:** #111, #108, #109
- **Tags:** game-design, planting, tree, decision-record

**Brief:** PlantingSpot is NEUTRAL — not typed by species. Original design: Spot remembers what tree was cut, only same species can be planted. Revised: any sapling can be planted in any spot. More fun.

**Details:**
- Original design (rejected):
  - Stump → PlantingSpot remembers TreeTypeData
  - Only sapling of SAME type can be planted there
  - "Realistic" but restrictive
- Revised design (current):
  - PlantingSpot is neutral (no treeTypeData field)
  - Sapling (PickupSapling) carries treeTypeData
  - Player can plant ANY species in ANY spot
  - More fun for "biome shift" gameplay (replace boring Spruce forest with valuable Birch trees)
- Implementation: treeTypeData injected via PickupSapling → PlantingSpot reads from sapling at plant time → passes to GrowingTree
- Lesson: realistic constraint ≠ fun. Iterate on player desires.

**Why it matters:** Design constraints should serve gameplay, not realism. Codified to prevent re-introduction of "realistic" restriction.

#### #111 ★ Adding new species = SO + models only (zero code changes)
- **Slug:** zero-code-changes-philosophy
- **Path:** genre/tycoon/decisions/zero-code-changes-philosophy.md
- **Type:** C
- **Template:** decision
- **Sources:** e55dbfd6-bb50
- **Related:** #108, #109, sweep2-1
- **Tags:** game-design, content-scaling, philosophy, scriptableobject

**Brief:** Adding a new tree species (or product, or worker role, or NPC type) = create new ScriptableObject + assign models + done. Zero code changes. Foundational philosophy of TT.

**Details:**
- New species workflow:
  1. Create folder `Assets/Models/Trees/Oak/`
  2. Create models: `Oak_Adult.fbx`, `Oak_Stump.fbx`, `Oak_Trunk_Fallen.fbx`, `Oak_Log.fbx`, `Oak_Sapling.fbx`, growth stages
  3. Right-click → Create → Timber Tycoon → Tree Type Data → TreeType_Oak
  4. Assign models to TreeType_Oak in Inspector
  5. Drop TreeType_Oak on a tree prefab variant. Done.
- Applied to: tree species, product types, worker roles, customer car variants
- Enabled by: ScriptableObject runtime injection (#108) + propagation chain (#109) + naming convention (sweep2-1) + GLOBAL_ROUTER (#121)
- Cost of philosophy: more SOs to maintain. Worth it for content scaling.
- Implication for future projects: ALWAYS design with "zero code per content variant" as constraint

**Why it matters:** Content scaling without code burden = the difference between "we have time to add 4 species" and "we ship with 12 species."

#### #124 LoadingStation decision (one point vs walking to racks)
- **Slug:** loading-station-vs-walking-decision
- **Path:** genre/tycoon/decisions/loading-station-decision.md
- **Type:** C
- **Template:** decision
- **Sources:** 03ec0ede-bc12
- **Related:** #125, #123
- **Tags:** game-design, qol, progression, loading

**Brief:** Loading crate considered: (A) walk to each rack manually OR (B) centralized LoadingStation. Decided: (A) early game (manual = gameplay), (B) late game upgrade (lvl 8). Best of both.

**Details:**
- Option A — Manual walking to each rack:
  - Pro: gameplay (player navigates sawmill, sees inventory directly)
  - Con: tedious late game (50 logs to load = 50 walks)
- Option B — Centralized LoadingStation:
  - Pro: efficient (one interaction)
  - Con: skips engagement, removes sense of "place"
- Decision: A first, B as upgrade
  - Early game: walk to each rack manually (gameplay focus)
  - Lvl 8 upgrade unlocks LoadingStation (QoL reward)
  - Player who upgrades feels rewarded ("I earned the convenience")
- LoadingStation.isUnlocked = false default, true after `OnUpgradePurchased` with id "loading_station"
- UI: one panel where player ticks "load N of product X" → system pulls from racks

**Why it matters:** Quality-of-life upgrades = perfect progression rewards. Player suffers manually → values automation. Always offer convenience as reward.

#### #138 ★ Multiplayer-from-MVP decision (NOT retrofit)
- **Slug:** multiplayer-from-mvp-decision
- **Path:** genre/survival/decisions/multiplayer-from-mvp-not-retrofit.md
- **Type:** C
- **Template:** decision
- **Sources:** bad9d002-2086 (Eskimo Simulator)
- **Related:** #066, #137
- **Tags:** game-design, multiplayer, architecture, networking, eskimo

**Brief:** Eskimo Simulator design decision: build multiplayer from MVP using FishNet + NetworkBehaviour from day 1, NOT retrofit SP code later. Retrofitting MP into SP codebase = rewriting most of game logic.

**Details:**
- Considered: ship SP first, add MP later (Subnautica path)
  - Pro: faster MVP
  - Con: SP code makes assumptions (singletons, instant updates, no rollback) that break with networking
  - Con: retrofitting MP = touching every system that has state
- Decision: MP from MVP
  - All gameplay systems built on NetworkBehaviour
  - Server-authoritative state (host-based listen server)
  - Even solo players run as host of 1 (same code path)
- Stack: Unity 6000.3.5f1 + URP + FishNet + ServiceLocator + GameEventSO + ISaveable + NetworkBehaviour
- Architecture co-exists with TT's stack (#066) — extension layer not replacement
- Implications: more upfront cost (~20% dev time), prevents existential refactor crisis

**Why it matters:** Cross-project lesson. If MP is in roadmap at all, design for it from MVP. Retrofitting MP is the #1 cause of indie studios failing to ship sequel.

#### #139 Chunk-based world loading (Eskimo Simulator)
- **Slug:** chunk-based-world-loading
- **Path:** genre/survival/patterns/chunk-based-world-loading.md
- **Type:** C
- **Template:** pattern
- **Sources:** bad9d002-2086
- **Related:** #138
- **Tags:** unity, world-streaming, chunks, survival, eskimo

**Brief:** Eskimo Simulator uses chunk-based world streaming (vs TT's single-scene approach) to support larger map (handcrafted hub-and-spoke with 6 progressive zones). Architectural divergence from TT.

**Details:**
- TT approach: single scene (`Demo_Scene.unity`), all content in memory
- Eskimo approach: world divided into chunks, load/unload based on player proximity
- Why chunks: 6 zones × 1km² each = 6km² total. Single-scene = ~3 minute load time + 8GB RAM
- Chunk size: 256m × 256m typical, 16 chunks visible at any time
- Streaming: Unity Addressables system, chunks as Addressable assets
- LOD per chunk: closer chunks full detail, distant chunks low-poly impostors
- Chunks define: terrain mesh, scattered props, ambient zones, NPC spawn points
- Save persistence: chunks save independently (player edits to one chunk don't bloat global save)

**Why it matters:** Different game scope = different architecture. TT can afford single-scene. Eskimo cannot. Pattern documented for future large-world projects.

#### #140 Audio strategy: minimal music + heavy ambient + voice bites
- **Slug:** audio-strategy-minimal-music-heavy-ambient
- **Path:** genre/cross-genre/decisions/audio-strategy-minimal-music-heavy-ambient.md
- **Type:** C
- **Template:** decision
- **Sources:** bad9d002-2086
- **Related:** #064, #089, #091
- **Tags:** game-design, audio, music, ambient, voice, cross-genre

**Brief:** Audio strategy for open-world / immersive games: minimal music (Valheim-style, only key moments), heavy ambient priority, ElevenLabs-generated voice bites in native languages for NPCs (Inuktitut for Eskimo, Polish for TT customers).

**Details:**
- Music: only at key moments (igloo built, niedźwiedź encounter, aurora event, story moments, main menu). Drums-of-Inuit motif. Pure ambient between music.
- Ambient: priority over music. Wind dynamic (intensity affects sound), kroki in snow per depth, trzask cienkiego lodu warning, qulliq buzzing, oddech in cold, wolves howling. Builds immersion 100x better than music.
- Voice: ElevenLabs-generated. Native languages for cultural authenticity (Inuktitut for Inuit NPCs in Eskimo Simulator). Short fragments (greetings, exclamations, emotional reactions) + text with subtitles for longer dialogue.
- Narrator (player internal monologue): English, key moments only ("I need to find shelter before nightfall.")
- Cross-applicability to TT: keep music ambient/light for sawmill, more music for city/customers
- Per audio category (#064): Music (low), Ambient (high), SFX (medium), UI (low), Voice (medium)

**Why it matters:** Music-saturated games feel "video gamey." Ambient-priority games feel atmospheric. Eskimo wants atmosphere. TT can benefit from same principle (sawmill = ambient over music).


---

### BATCH 7 — workflow/claude-code + mcp-tools (entries #015-#137)

#### #015 Scene attachment check before deleting MonoBehaviour
- **Slug:** scene-attachment-check-before-deleting-monobehaviour
- **Path:** workflow/claude-code/scene-attachment-check-before-deleting.md
- **Type:** D
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #013, #014, #038
- **Tags:** claude-code, unity, scene-safety, deletion

**Brief:** Before deleting a MonoBehaviour script file, Claude Code MUST search all scenes/prefabs for script references. Deleting in-use scripts leaves scene with "missing script" errors, manual cleanup required.

**Details:**
- Pre-delete checklist:
  1. `grep -r "ClassName" Assets/Project/Scripts/*.cs` — find code references
  2. `grep -r "ClassName" Assets/**/*.unity Assets/**/*.prefab` — find scene/prefab references
  3. Open `.asset` ScriptableObject files: search for class name in YAML
  4. ONLY if all clear: delete
- Recovery if missing: see #014 (manual Inspector Remove Component)
- Coplay MCP / Code agents: implement this check as automated step

**Why it matters:** "Delete refactored code" is one-click; "fix 47 missing scripts in scene" is half a day. Asymmetric cost.

#### #016 Backup scene before modify
- **Slug:** backup-scene-before-modify
- **Path:** workflow/claude-code/backup-scene-before-modify.md
- **Type:** D
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #013, #014
- **Tags:** claude-code, unity, backup, scene-safety

**Brief:** Before any structural scene modification by Code or Coplay agents, copy scene file to `Assets/_Backup_{YYYY-MM-DD}/` for rollback.

**Details:**
- Copy command: `cp Assets/Demo_Scene.unity Assets/_Backup_2026-05-17/Demo_Scene.unity`
- Create date-stamped folder per session
- Backup BEFORE: bulk add/remove components, deleting GameObjects, restructuring hierarchy
- Don't need backup for: minor Inspector tweaks (Unity handles undo)
- Keep last 7 days of backups, prune older with `find Assets/_Backup_* -mtime +7 -delete` periodically
- Git also backs up but is slower to query and depends on remembering to commit

**Why it matters:** Defense in depth. Git + scene backups + Unity undo = three safety nets. Costs 1 second per backup, saves hours per disaster.

#### #028 ★ Context degradation threshold (20-40% remaining)
- **Slug:** context-degradation-threshold
- **Path:** workflow/claude-code/context-degradation-threshold.md
- **Type:** A
- **Template:** lesson
- **Sources:** bccf249b-ec6b, 6967a837-dc86
- **Related:** #032, #034
- **Tags:** claude-code, context-window, prompt-strategy

**Brief:** Claude Code performance degrades visibly when context is between 20-40% remaining. Above 40% = full capability. Below 20% = unreliable. Plan `/clear` before falling into the danger zone.

**Details:**
- Indicators of degradation:
  - Re-reads files it already has in context
  - Asks about previously-stated facts
  - Suggests solutions that conflict with prior decisions
  - Misses simple edits ("forgot to remove old block")
- Threshold rule: at 40% remaining, evaluate "should I /clear soon?"
- Below 20%: stop work, /compact or /clear, lose less productivity than degraded output
- Why: model attention spreads thin, recent context drowns out older critical instructions
- Mitigations: shorter prompts, single-task per /clear cycle, write findings to disk before /clear

**Why it matters:** "Why is Code suddenly bad?" — usually context exhaustion, not model issue. Recognize early, /clear preemptively.

#### #029 ★ Console Collapse → unique [#{i}] suffix in loop logs
- **Slug:** console-collapse-loop-suffix
- **Path:** workflow/claude-code/console-collapse-loop-suffix.md
- **Type:** A
- **Template:** lesson
- **Sources:** bccf249b-ec6b
- **Related:** #028
- **Tags:** unity, debug-log, console, collapse, claude-code

**Brief:** Unity Console "Collapse" mode merges identical log strings. In loops, this hides per-iteration data. Fix: include unique suffix `[#{i}]` per loop iteration.

**Details:**
- Symptom: loop prints 10 logs, Console shows "Same message" — actual data invisible
- Root cause: Unity Console Collapse mode (default ON) merges by exact string match
- Fix in code:
  ```csharp
  for (int i = 0; i < items.Count; i++) {
      Debug.Log($"[#{i}] Processing {items[i].name}");
  }
  ```
- Now each log unique → all appear in Console even with Collapse on
- Alternative: turn off Collapse (icon in Console toolbar) — but easy to forget
- Standard convention: ALL loop logs in TT include `[#{i}]` or other unique marker

**Why it matters:** "Why is my loop only processing item 3?" — it isn't, Console is hiding the others. 5-minute waste prevented by 1-second convention.

#### #030 ★ Stop hook = infinite loop risk, remove
- **Slug:** stop-hook-infinite-loop-risk
- **Path:** workflow/claude-code/stop-hook-infinite-loop-risk.md
- **Type:** A
- **Template:** lesson
- **Sources:** (memory — April 2026 Claude Code config)
- **Related:** #031, #034
- **Tags:** claude-code, hooks, configuration, infinite-loop

**Brief:** Claude Code Stop hook (auto-continue when context full) was tested and removed. Caused infinite loops on tasks Code couldn't complete. Manual `/continue` is safer.

**Details:**
- Stop hook config: in `.claude/settings.json`, runs script on context-full event
- Tested behavior: Code hits context cap → Stop hook triggers → spawns new Code instance → same context cap → repeat → infinite
- Particular risk: when Code is stuck on unsolvable problem (race condition that doesn't reproduce)
- Decision: removed from TT's `.claude/` config
- Replacement: Hunter manually invokes `/continue` after evaluating progress
- Lesson generalizable: automation that doesn't have stop condition = potential infinite loop

**Why it matters:** Two cases of Stop hook running overnight, accumulating Claude credits. Removed = safety.

#### #032 /clear vs /compact decision rules
- **Slug:** clear-vs-compact-decision-rules
- **Path:** workflow/claude-code/clear-vs-compact-decision-rules.md
- **Type:** D
- **Template:** lesson
- **Sources:** (memory)
- **Related:** #028, #034
- **Tags:** claude-code, context-management, clear, compact

**Brief:** `/clear` = full reset (use when starting unrelated task). `/compact` = summarize current context (use to continue same task with smaller footprint). 2 terminals max parallel.

**Details:**
- `/clear` use cases:
  - New unrelated task
  - Previous task complete, moving to next
  - Context heavily degraded (see #028)
- `/compact` use cases:
  - Mid-task, need more headroom
  - Don't want to lose current task state
  - Approach completion, just need final iterations
- 2 terminals max: 1 for primary task, 1 for emergency (e.g., asking for diagnostic info while main terminal runs build)
- 3+ terminals: too much cognitive load, easy to lose track of which is which
- Hunter's convention: always state `/clear: TAK` or `/clear: NIE — kontynuacja` with reasoning at top of each new prompt

**Why it matters:** Context management is the difference between productive Code session and "Code is confused" session.

#### #034 3-level analysis system
- **Slug:** three-level-analysis-system
- **Path:** workflow/claude-code/three-level-analysis-system.md
- **Type:** D
- **Template:** lesson
- **Sources:** (memory — April 2026 ecosystem)
- **Related:** #033, #036
- **Tags:** claude-code, analysis, workflow, decision-process

**Brief:** Claude Code tasks classified by analysis depth: Level 1 (full analysis + accept), Level 2 (brief justification + accept), Level 3 (immediate). Avoid over-thinking simple tasks, avoid under-thinking complex ones.

**Details:**
- Level 1 (5-10 min thinking): architectural changes, new features touching multiple systems, anything affecting save format
- Level 2 (1-2 min thinking): bug fixes with clear cause, single-file edits with non-obvious implications
- Level 3 (immediate): obvious changes (typo fixes, value adjustments, formatting)
- Hunter signals level in prompt: `Level 1:` / `Level 2:` / `Level 3:` prefix
- Or Code infers: complex prompt = Level 1 default; simple find/replace = Level 3 default
- Time budget per level: helps Code allocate attention appropriately

**Why it matters:** "Should I deeply analyze this?" — Level system answers. Saves time on simple, prevents shallow on complex.

#### #036 ★ Quad-backtick Claude Code prompt format
- **Slug:** quad-backtick-claude-code-prompt-format
- **Path:** workflow/claude-code/quad-backtick-prompt-format.md
- **Type:** D
- **Template:** lesson
- **Sources:** (memory — Hunter's prompt convention)
- **Related:** #034, #143
- **Tags:** claude-code, prompt-format, syntax, convention

**Brief:** Hunter's standard prompt format for Claude Code: wrapped in QUADRUPLE backticks, starts with `# TASK: <title>`, uses `Find: / Replace with:` blocks for exact edits, ends with "Do NOT modify any other files."

**Details:**
- Outer wrapper: ````` (4 backticks) — allows nested code blocks with triple backticks inside
- Header: `# TASK: <short title>`
- Edit blocks:
  ```
  Find:
  <exact text from file>
  
  Replace with:
  <new text>
  ```
- File references: full path `Assets/Project/Scripts/SomeFile.cs`
- Bounded scope: end with "Do NOT modify any other files."
- /clear decision (CRITICAL — separate code block from rest of prompt):
  ```
  /clear: TAK — fresh start, nowy temat
  ```
  THEN main task in a SEPARATE block.
- Hunter NEVER puts /clear in same block as task — Code can't accept them as one command.

**Why it matters:** Format consistency = Code parses prompts reliably. Variations = Code interprets wrong, edits wrong file, or skips /clear.

#### #080 "DO NOT MOVE" hardcoded positions convention
- **Slug:** do-not-move-hardcoded-positions
- **Path:** workflow/claude-code/do-not-move-hardcoded-positions.md
- **Type:** A
- **Template:** lesson
- **Sources:** (memory — map design conventions)
- **Related:** #078, #079, #126
- **Tags:** claude-code, scene, positions, convention, level-design

**Brief:** Scene objects with positions tuned by designer get comment `// DO NOT MOVE` next to position values. Code agents must respect this — never auto-adjust positions of marked objects.

**Details:**
- Convention in C# scripts: comment near hardcoded Vector3 positions
  ```csharp
  // DO NOT MOVE — manually positioned by Hunter
  Cliff_Waterfall.transform.position = new Vector3(184.5f, 7.2f, -82.3f);
  ```
- Setup scripts (e.g., RebuildFullMap.cs) include comment block listing locked positions
- Coplay MCP agents instructed to NEVER modify lines marked with this comment
- Applies to: cliff, waterfall, bridge, cave entrance, individual mountain positions, sawmill placement
- Why hardcoded: heightmap can't represent these (see #078) + designer carefully placed for visual composition
- Counter-example: enemy spawn points, NPC parking slots — DO move (designer expects auto-balancing)

**Why it matters:** "Code regenerated the map and now my cliff is in the river" — DO NOT MOVE prevents this category of bug.

#### #126 Intentionally LOW maxCapacity for test racks
- **Slug:** intentionally-low-maxcapacity-test-racks
- **Path:** workflow/claude-code/intentionally-low-maxcapacity-test-racks.md
- **Type:** A
- **Template:** lesson
- **Sources:** acb055d2-01aa
- **Related:** #096+119
- **Tags:** unity, testing, iteration, capacity, debug

**Brief:** During development, use LOW maxCapacity values for test racks (Plank=10, Bark=4) to hit "rack full" edge case quickly. Production values come later.

**Details:**
- Production maxCapacity (planned): Plank rack 50, Bark rack 30
- Test phase values: Plank 10, Bark 4 — fills in 30 seconds of gameplay
- Why: hitting full-rack edge cases in production values = 10 minutes of repetitive gameplay per test
- Marker in code/SO: comment "INTENTIONALLY LOW for testing" — flag to revert before ship
- Pre-launch checklist: search comment, update to production values
- General pattern: design for testability, mark debug values clearly

**Why it matters:** Test fast, iterate fast. Production values come last. Marker prevents shipping with debug values.

#### #136 Universal cleanup post-migration
- **Slug:** universal-cleanup-post-migration
- **Path:** workflow/claude-code/universal-cleanup-post-migration.md
- **Type:** A
- **Template:** lesson
- **Sources:** dab6f299-7ec0
- **Related:** #133, #135, #042
- **Tags:** claude-code, cleanup, debug-code, migration, technical-debt

**Brief:** After replacing debug shortcuts with real interactions (e.g., `#if UNITY_EDITOR U key` → real kiosk), remove the debug code. Zero stale debug code in production.

**Details:**
- Common pattern: debug shortcut implemented early ("U key opens upgrade shop") for testing
- Migration: real interaction (kiosk) replaces shortcut
- Cleanup step often skipped: leaves debug shortcut active alongside real interaction
- Anti-pattern instances:
  - Cheat keys persisting in shipped builds
  - Test data exposed in real UI
  - Debug menus accessible to players
- Convention: at sprint end, grep for `#if UNITY_EDITOR` and `// TODO debug` — review each, remove if obsolete
- Migration sprint markers (see #037 + #133): explicitly note "remove U shortcut" as cleanup task

**Why it matters:** "I shipped my game with cheat codes accidentally" — embarrassing. Cleanup discipline prevents it.

#### #137 Cross-project consistency lesson (Eskimo using identical stack)
- **Slug:** cross-project-consistency-stack-reuse
- **Path:** workflow/claude-code/cross-project-stack-reuse.md
- **Type:** D
- **Template:** lesson
- **Sources:** bad9d002-2086
- **Related:** #066, #138
- **Tags:** claude-code, architecture, reuse, eskimo, multiproject

**Brief:** Eskimo Simulator uses identical core stack to TT (Unity 6000.3.5f1 + URP + ServiceLocator + GameEventSO + ISaveable). Known toolchain = same workflow, transferable skills, faster Eskimo MVP.

**Details:**
- Shared stack:
  - Unity version 6000.3.5f1 (lock to TT's tested version)
  - URP render pipeline
  - ServiceLocator + GameEventSO + ISaveable patterns
  - Claude Code as primary dev pipeline
  - Blender for modeling, ElevenLabs for audio
- Eskimo additions:
  - FishNet networking layer (new — see #138)
  - NetworkBehaviour layer on top of existing patterns
  - Addressables for chunk streaming (#139)
- Anti-pattern avoided: "different game = different stack" — would force re-learning, re-building tooling, re-establishing conventions
- Benefit: existing CLAUDE.md / .claude/ ecosystem transfers (~80% reusable)
- Cost: limited to validated tech (no experimenting with Unreal mid-Eskimo)

**Why it matters:** Cross-project consistency = compounding skill investment. Each project doesn't start from zero, builds on toolchain mastery.

#### #143 Iterative checkpoint workflow for complex generated assets
- **Slug:** iterative-checkpoint-workflow-generated-assets
- **Path:** workflow/claude-code/iterative-checkpoint-workflow.md
- **Type:** C
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #141, #142
- **Tags:** claude-code, workflow, generated-assets, checkpoints, blender

**Brief:** Complex assets (PlankMaker machine: frame → bed → cutting head → control panel → details) generated in iterative checkpoints. Each checkpoint requires Hunter approval before next. No jump-ahead.

**Details:**
- Workflow per checkpoint:
  1. Code generates partial asset (e.g., just the frame)
  2. Renders multi-angle preview (isometric + top + front)
  3. Hunter reviews preview, says ✅ / 🚫 / 🔧 (approve/reject/fix)
  4. On ✅: proceed to next checkpoint
  5. On 🔧: Code fixes, regenerates preview, back to step 3
  6. On 🚫: rethink approach (rare)
- Per asset: 4-6 checkpoints typical
- Why: errors caught early are cheap; errors caught late require redoing everything downstream
- Avoid: "generate full machine in one shot" — accumulates errors, hard to localize problem
- Combine with #142 (ZERO floating mandate) per checkpoint

**Why it matters:** Iterative validation = bounded error. One-shot generation = unbounded error.

#### #031 MCP wildcard permissions format
- **Slug:** mcp-wildcard-permissions-format
- **Path:** workflow/mcp-tools/mcp-wildcard-permissions-format.md
- **Type:** A
- **Template:** lesson
- **Sources:** (memory — Claude Code config April 2026)
- **Related:** #030
- **Tags:** claude-code, mcp, permissions, configuration

**Brief:** MCP permissions in `.claude/settings.json` use wildcard format `mcp__{server}__*` (double underscore). Specific tool format `mcp__{server}__{tool}` works but wildcards are cleaner.

**Details:**
- Wildcard syntax: `mcp__blender-mcp__*` (matches all blender-mcp tools)
- Specific: `mcp__blender-mcp__create_object`
- Why double underscore: MCP protocol convention (single underscore reserved)
- Common servers in TT: `blender-mcp`, `coplay-mcp`, `elevenlabs-mcp`, `screen-capture-mcp`
- Permission file: `.claude/settings.json` → `permissions.allowed_tools`
- Pre-April 2026: used various inconsistent formats (single underscore, no prefix). April fix consolidated to `mcp__server__*`

**Why it matters:** Wrong format = MCP server doesn't connect, no error message. Fix = update format, restart Code, works.

#### #035 Skill loading on-demand vs `@` reference trade-off
- **Slug:** skill-loading-on-demand-vs-reference
- **Path:** workflow/mcp-tools/skill-loading-on-demand-vs-reference.md
- **Type:** A
- **Template:** lesson
- **Sources:** (memory — skill ecosystem)
- **Related:** #034, #143
- **Tags:** claude-code, skills, context-budget, on-demand

**Brief:** Skills can be loaded `@skillname` (full skill in context immediately) OR on-demand (Code searches when relevant). On-demand = saves context budget; `@` = guaranteed loaded.

**Details:**
- `@skill` reference: skill file read into context on prompt, costs tokens upfront
- On-demand: Code uses tool_search to find skill when topic comes up, costs round-trip
- Use `@` for: skills definitely needed in current session (active development), small skills (<200 lines)
- Use on-demand for: large skills, skills "might or might not" be relevant, multi-skill scenarios
- TT convention: `@unity-conventions` always (small, always relevant). `@blender-export` on-demand (large, episodic relevance).
- Skill design tip: small + focused skills load cheaply on demand; large all-in-one skills cost too much

**Why it matters:** Context budget = developer productivity. Skill loading strategy = 20-30% context savings if optimized.


---

### BATCH 8 — workflow/3d-models + asset-pipeline (entries #007-#142)

#### #007 Tripo "firewood log" misinterpretation — vocab fix
- **Slug:** tripo-firewood-log-vocab
- **Path:** workflow/3d-models/tripo-vocab-firewood-wedge.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals — Tripo workflow)
- **Related:** #008, #009
- **Tags:** tripo, ai-3d, prompt-engineering, vocabulary

**Brief:** Tripo AI interprets "firewood log" as full tree trunk. For TT firewood pieces, use "wedge-shaped piece of wood, triangular cross section, quarter of a log."

**Details:**
- Bug: prompt "firewood log" → Tripo generates 3m cylinder (tree trunk shape)
- Expected: small wedge-shaped piece (TT firewood = product to sell, not transport object)
- Vocab translation table built through trial:
  - "Firewood" → "wedge-shaped piece, triangular cross section, quarter of a log"
  - "Plank" → "rectangular wooden board, flat, smooth surface"
  - "Bark" → "tree bark fragment, curved, fibrous outer layer"
  - "Pellet bag" → "small sealed sack of compressed wood pellets"
- General lesson: Tripo trained on common interpretations — niche terminology needs description-first
- Time saved: trial-and-error with each new prompt vs. consulting vocab table

**Why it matters:** Each Tripo generation costs minutes. Wrong vocab = wasted credits + time. Vocab table = first-try success rate ~80%.

#### #128 ★ Procedural textures in Blender Cycles (commercial release rationale)
- **Slug:** procedural-textures-cycles-commercial
- **Path:** workflow/3d-models/procedural-textures-cycles-commercial.md
- **Type:** A
- **Template:** lesson
- **Sources:** fda34b50-7e59
- **Related:** #001, #003
- **Tags:** blender, cycles, procedural, commercial, licensing

**Brief:** For commercial release (TT will ship on Steam), use procedural textures (Wave + Voronoi + Noise + ColorRamp → Principled BSDF) — ZERO external image files = zero licensing concerns.

**Details:**
- Asset license risk: PBR textures from sites (CC0, royalty-free) often have terms requiring attribution or non-commercial use
- Procedural materials: 100% generated in Blender, no third-party assets
- Standard procedural setup for low-poly:
  - **Wood:** Wave Texture (Bands, scale 5) → ColorRamp (brown gradient) → Principled BSDF (Roughness 0.8)
  - **Concrete:** Noise Texture (scale 8) + Voronoi (scale 6) → Mix → ColorRamp (gray with variation) → Principled BSDF
  - **Bark:** layered Noise + Voronoi → bumpy ColorRamp → Bump output
  - **Metal:** Voronoi (Distance to Edge, scale 20) + Noise (subtle) → ColorRamp → Principled BSDF (Metallic 0.9, Roughness 0.3)
- Bake to PNG before export (#001) — procedural shaders don't survive FBX
- Bonus: file size of Blender file with procedural = small (no embedded PNGs)
- For TT: all materials procedural until proven need for image-based (none so far)

**Why it matters:** "Did you check the license of that texture?" — never need to. Procedural = free of all third-party constraints.

#### #141 ★ Blender headless Python script generation pattern
- **Slug:** blender-headless-python-generation
- **Path:** workflow/3d-models/blender-headless-python-generation.md
- **Type:** A
- **Template:** pattern
- **Sources:** dab6f299-7ec0
- **Related:** #142, #143
- **Tags:** blender, python, headless, automation, generation

**Brief:** Generate 3D assets programmatically via Blender headless mode: `blender.exe --background --python script.py`. Script creates mesh, sets materials, saves .blend + exports FBX, renders preview.

**Details:**
- Command pattern:
  ```bash
  "C:/Program Files/Blender Foundation/Blender 5.0/blender.exe" \
    --background \
    --python "<project-root>/_BlenderScripts/01_create_frame.py"
  ```
- Script structure:
  1. Clear scene (delete default cube + lights, keep camera + world)
  2. Set units to Metric/Meters
  3. Create geometry (mesh primitives + transforms)
  4. Assign materials (slot names match Unity material naming convention)
  5. Save .blend file
  6. Export FBX with standard settings (#005)
  7. Render multi-angle preview (isometric/top/front) per #143
- Output paths convention:
  - `_BlenderScripts/` — Python scripts (version-controlled)
  - `_BlenderOutputs/{AssetName}/` — generated .blend + .fbx + preview PNGs
- Caveat: Blender Python API can differ between versions — script targets Blender 5.0+

**Why it matters:** Manual Blender modeling is slow + non-reproducible. Headless scripts = generated assets deterministically, version-controllable.

#### #142 ★ ZERO floating elements / ZERO flickering mandate
- **Slug:** zero-floating-zero-flickering-mandate
- **Path:** workflow/3d-models/zero-floating-zero-flickering-mandate.md
- **Type:** A
- **Template:** lesson
- **Sources:** dab6f299-7ec0
- **Related:** #141, #143
- **Tags:** blender, modeling, quality, z-fighting, mandate

**Brief:** Hunter's mandate for ALL generated assets: ZERO floating elements (visible gap between adjacent surfaces), ZERO Z-fighting flickering (coplanar surfaces). Physical connection mandatory.

**Details:**
- Failure mode 1: floating — beam appears 0.05m above pillar surface (visible gap)
- Failure mode 2: flickering — two surfaces at exact same Z position, GPU oscillates between them
- Fixes:
  - Surfaces meant to touch: same coordinate exactly (e.g., pillar top at Z=1.8, beam bottom at Z=1.8 — overlapping, not stacked)
  - Surfaces near each other: offset by 0.001m intentionally (prevents Z-fighting)
- Visual sanity check per checkpoint render (#143)
- Per Blender Python script: assert position consistency before save
  ```python
  assert abs(pillar_top_z - beam_bottom_z) < 0.001, "Z mismatch!"
  ```
- Common violation: copy-paste position values introduces rounding drift

**Why it matters:** Floating gaps = "looks bad and unfinished." Flickering = "looks broken." Both shipped = player refunds. Mandate prevents.

#### #009 Tripo cleanup pipeline
- **Slug:** tripo-cleanup-pipeline
- **Path:** workflow/asset-pipeline/tripo-cleanup-pipeline.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals — Tripo workflow)
- **Related:** #007, #008, #010
- **Tags:** tripo, blender, cleanup, decimation, retopo

**Brief:** Tripo AI generates 3D models with issues: inner faces (hidden geometry), broken UVs, 8k+ tris (overbudget). Standard cleanup pipeline: remove inner faces → UV rework → bake to PNG → decimate to 5k tris.

**Details:**
- Step 1: import .obj/.glb from Tripo into Blender
- Step 2: identify and delete inner faces (View → Select Internal Faces, then delete)
- Step 3: recalculate normals (Mesh → Normals → Recalculate Outside)
- Step 4: UV rework — Smart UV Project with minimal padding for static props, manual unwrap for hero assets
- Step 5: bake current material to PNG (Cycles bake, Diffuse + Normal + AO maps)
- Step 6: decimate to target tri budget (TT: 5000 tris for hero props, 2000 for background)
- Step 7: export as FBX with standard settings (#005)
- Time: 30-60 minutes per asset (vs hours to model from scratch)

**Why it matters:** Raw Tripo output = unusable in Unity. Cleanup pipeline = production-ready in <1 hour.

#### #010 Tripo asymmetric / floating elements → Blender retopo
- **Slug:** tripo-asymmetric-floating-retopo
- **Path:** workflow/asset-pipeline/tripo-asymmetric-floating-retopo.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #009, #142
- **Tags:** tripo, blender, retopo, asymmetry, cleanup

**Brief:** Tripo often generates asymmetric models (left/right wheels different) or floating elements (loose details unattached to main mesh). Fix in Blender via mirror + snap + merge by distance.

**Details:**
- Asymmetry fix:
  1. Delete bad-symmetry side (e.g., left half if right is good)
  2. Add Mirror modifier (X-axis), apply
  3. Clean centerline if duplicate vertices (Mesh → Merge by Distance, 0.001m threshold)
- Floating elements fix:
  1. Identify in viewport (often visible as disconnected pieces during rotate)
  2. Edit Mode → select disconnected mesh → Snap to nearest face on main mesh
  3. Mesh → Merge by Distance to weld snapped vertices
- Time investment: 10-30 minutes per model, depending on issues
- When to skip: low-budget background props (player won't notice asymmetry at distance)

**Why it matters:** Tripo's strengths are speed + concept; symmetry is a known weakness. 20-minute manual fix turns "AI slop" into "production asset."

#### #065 Audio asset pipeline (ElevenLabs + Suno MCP + FFmpeg normalize)
- **Slug:** audio-asset-pipeline-elevenlabs-suno
- **Path:** workflow/asset-pipeline/audio-asset-pipeline.md
- **Type:** A
- **Template:** pattern
- **Sources:** a0ce5046-a69a, (memory — audio workflow)
- **Related:** #064, #140
- **Tags:** elevenlabs, suno, ffmpeg, audio, pipeline

**Brief:** Audio asset pipeline for TT: ElevenLabs MCP for SFX + voice, Suno (no MCP, manual upload) for music, FFmpeg normalize all to -16 LUFS. Audio formats: WAV for production, OGG for runtime.

**Details:**
- ElevenLabs MCP: generate SFX via prompt (e.g., "axe chopping wood, mid-frequency thunk")
- Suno: generate music via web interface (no MCP available), download .mp3
- Normalize all to -16 LUFS via FFmpeg:
  ```bash
  ffmpeg -i input.wav -af "loudnorm=I=-16:LRA=11:TP=-1.5" output.wav
  ```
- Convert to OGG for runtime: `ffmpeg -i normalized.wav -c:a libvorbis -q:a 4 final.ogg`
- Folder structure:
  - `Assets/Audio/SFX/` — sound effects, named `sfx_kategoria_opis.ogg`
  - `Assets/Audio/Music/` — music tracks, `music_kategoria_opis.ogg`
  - `Assets/Audio/Ambient/` — ambient loops, `ambient_zone_opis.ogg`
  - `Assets/Audio/UI/` — UI sounds
  - `Assets/Audio/Voice/` — NPC voice bites
- Audio Mixer routes (see #064)

**Why it matters:** Unnormalized audio = "this clip is way louder than others." Normalization once = consistent volume always.

#### #081 Polybrush iteration rule: never go back to regenerator
- **Slug:** polybrush-iteration-rule
- **Path:** workflow/asset-pipeline/polybrush-iteration-rule.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals — terrain workflow)
- **Related:** #078, #082
- **Tags:** unity, polybrush, terrain, iteration, workflow

**Brief:** After exporting terrain mesh from LowPolyTerrainGenerator and starting Polybrush sculpting, NEVER return to regenerator. Regen wipes all Polybrush edits.

**Details:**
- Workflow:
  1. LowPolyTerrainGenerator: generate base terrain via custom editor (#060)
  2. Export mesh as asset (`AssetDatabase.CreateAsset`)
  3. Polybrush sculpting: bumps, hollows, hand-painted vertex colors
  4. Save asset
- Critical rule: step 4 is IRREVERSIBLE WRT step 1. Going back to step 1 regenerates terrain, overwrites mesh asset, loses all Polybrush work.
- Recovery: only via version control (git revert) — Polybrush has no undo across sessions
- Convention: after Polybrush starts, terrain parameters in regenerator should NOT change for that scene
- For experiments: use separate scenes/branches

**Why it matters:** Hours of Polybrush work lost from one "let me tweak the noise scale" click. Codify rule, prevent disaster.

#### #082 Polybrush settings for low-poly terrain
- **Slug:** polybrush-settings-low-poly
- **Path:** workflow/asset-pipeline/polybrush-settings-low-poly.md
- **Type:** A
- **Template:** lesson
- **Sources:** (from 42-originals)
- **Related:** #081, #058
- **Tags:** unity, polybrush, terrain, settings, low-poly

**Brief:** Polybrush settings for TT low-poly terrain: Outer Radius 40, Sculpt Power 30, Falloff Curve linear, Strength 1.0. Higher values = "soft" bumps that fit aesthetic.

**Details:**
- Outer Radius: 40 (large brush — sculpts smooth low-frequency variation, not detail)
- Inner Radius: 20 (50% of outer — natural falloff)
- Sculpt Power: 30 (moderate height change per click)
- Falloff Curve: Linear (no abrupt edges — fits low-poly soft style)
- Strength: 1.0 (full effect per stroke)
- Other tools: Smooth (radius 30, strength 0.5) to soften harsh transitions
- Vertex color paint: separate Polybrush tool, different settings (radius 20, soft falloff)
- After Polybrush: use Mesh → Smooth Vertex (custom Editor tool) for additional softening

**Why it matters:** Default Polybrush settings produce sharp bumps incompatible with TT low-poly aesthetic. Tuned settings = bumps that fit the style.


---

## 8. MOC.md Template (Code generates this)

After all batches complete, Code creates `<kb-root>\MOC.md` with this structure (replace `{path}` with actual paths from entries):

```markdown
# MOC — Map of Content

Last updated: {timestamp}
Total entries: ~150

## Engine

### Lessons (technical know-how, gotchas)
- [Procedural textures must be baked before FBX export]({path})
- [bake_space_transform + linked duplicates = 90° rotation bug]({path})
- [FBX export standard settings (Blender → Unity)]({path})
- [Asset origin at bottom-center convention]({path})
- [MeshExporter OBJ pitfalls (3 critical bugs)]({path})
- [URP shadow cascade tuning for low-poly terrain]({path})
- [NEVER save_scene or DestroyImmediate in Play Mode]({path})
- [Forward axis = -transform.right (Blender FBX quirk)]({path})
- [Self-collision compound BoxColliders → Physics.IgnoreCollision]({path})
- [Minimum turnFactor 0.3 for low-speed arcade steering]({path})
- [Vertex color gamma correction Blender → Unity]({path})
- [Desaturated colors for low-poly aesthetic]({path})
- [Separate-objects mapping rule (heightmap limitations)]({path})
- [Runtime vs Editor script separation]({path})
- [Editor Scene View input capture]({path})
- [CapsuleCollider direction axis cheatsheet]({path})
- [Cylindric vs rectangular beams visual contrast]({path})
- [Read actual code before hypothesizing (debugging discipline)]({path})
- [Tag assignment: code vs Inspector]({path})

### Patterns (reusable techniques)
- [Vehicle interaction zones as triggers]({path})
- [Awake-init for ISaveable with dependencies]({path})
- [GetOrAddComponent extension method]({path})
- [Storage migration: Primary new + Legacy fallback]({path})
- [Before-delete legacy class checklist]({path})
- [ScriptableObject runtime-assigned references]({path})
- [Diegetic 3D button raycast pattern]({path})
- [Camera lock: save → lerp → restore]({path})
- [Sliding head bandsaw mouse-drag tempo minigame]({path})
- [MaterialPropertyBlock for runtime color variants]({path})
- [Quest highlight pattern (quest-flag mechanism)]({path})
- [River mesh semi-elliptical cross-section]({path})
- [4-phase weighted smoothstep day/night transition]({path})
- [Procedural skybox sun/moon trick]({path})
- [Custom Editor pattern for generators]({path})
- [Tool viewmodel as child of camera]({path})
- [AudioManager + Mixer architecture]({path})
- [Parallel architecture pattern (Locator + Events + ISaveable + Singleton)]({path})
- [GameEventSO event channel pattern]({path})
- [ISaveable contract]({path})
- [GameStateMachine pattern]({path})
- [VFX performance budget]({path})
- [VFX trigger pattern via GameEventSO]({path})
- [Mountains hierarchy (front + backdrop)]({path})
- [Cliff + Waterfall hidden cave]({path})
- [Audio Mixer snapshots per game state]({path})
- [Footstep raycast surface detection]({path})
- [Audio occlusion (raycast + LPF + volume)]({path})
- [AudioReverbZone per environment]({path})
- [Ambient crossfade zone-based]({path})
- [Typography & accessibility stack]({path})
- [Shared mesh + materials reference (no duplication)]({path})
- [Rack visual fill alignment]({path})
- [Catmull-Rom spline road mesh]({path})
- [FlattenTerrainUnderRoad smoothstep]({path})
- [MeshCollider on roads (stackable)]({path})
- [ScriptableObject runtime injection]({path})
- [SO propagation chain via parameter passing]({path})
- [Trunk fall physics config]({path})
- [VehicleCamera runtime attach/detach]({path})
- [Vehicle enter/exit choreography]({path})
- [Universal camera lock (canMove flag)]({path})
- [StatisticsManager pattern]({path})
- [StorageRackRegistry singleton + auto-register]({path})
- [GLOBAL_ROUTER storage pattern]({path})
- [Dictionary warehouse registry]({path})
- [Storage activation gating via upgrade]({path})
- [CrateManager tier progression]({path})
- [Architectural elements naming convention]({path})
- [Collider distribution rule]({path})
- [TreeState + StumpState enums]({path})
- [Migration pattern with rollback safety]({path})
- [ReputationLevels data-driven progression]({path})
- [KioskInteractable + Cube placeholder]({path})
- [ChoppableTree multi-type naming convention]({path})

### Anti-Patterns (what NOT to do)
- [Cycles bake for solid colors → black/blurred]({path})
- [Scene files are binary, never text-edit]({path})
- [Legacy code conflict after refactor]({path})
- [Rotating Directional Light → black terrain at dawn/dusk]({path})
- [Script overrides prefab Inspector values (CapsuleCollider case)]({path})
- [Script overrides DirtClump Inspector values]({path})
- [Low-poly water side-wave anti-pattern]({path})
- [Race condition: Start() vs Instantiate parameter]({path})

## Genre

### Tycoon

#### Patterns
- [NPC parking PD controller]({path})
- [Pipeline-style NPC spawn (purchase-triggered)]({path})
- [Initial fill on load (don't serialize NPC state)]({path})
- [NavMesh + kinematic waypoints hybrid]({path})
- [Object pooling NPCs + FIFO queue]({path})
- [Carry capacity progression + sprint advantage]({path})
- [Multi-step quest checklist]({path})
- [Customer tier system (Regular/Contractor/VIP)]({path})
- [WaterZone gameplay component]({path})
- [Minecraft-style lighting]({path})
- [ToolWheel UX pattern]({path})
- [WorkerData blueprint + WorkerInstance runtime split]({path})
- [Worker SimulateWorkCycle abstraction (no NavMesh)]({path})
- [Worker output quality distribution]({path})
- [OrderFulfiller interface (asymmetric player vs NPC)]({path})
- [Wing snap-points modular fade-in]({path})
- [Debris cleanup single-click drop]({path})
- [StorageRack family system]({path})
- [Visualization ratio (10:1 inventory to visual)]({path})
- [Top-down camera minigame (stump digging)]({path})

#### Decisions
- [Sales flow: hybrid player + NPC worker]({path})
- [Tool tier replacement (not multiple in inventory)]({path})
- [Quantity-not-quality design principle]({path})
- [VFX wycofane (sawdust/kurz/liście cut from MVP)]({path})
- [Building progression instant spawn (Tavern Manager style)]({path})
- [Player-built vs purchased dichotomy]({path})
- [UI Phase 4 catalog roadmap]({path})
- [Rack architecture decision (3 options analyzed)]({path})
- [PlantingSpot universal-not-typed design]({path})
- [Zero code changes per species philosophy]({path})
- [LoadingStation: walking vs centralized (QoL upgrade)]({path})

### Survival (Eskimo Simulator cross-references)

#### Patterns
- [Chunk-based world loading]({path})

#### Decisions
- [Multiplayer-from-MVP (not retrofit)]({path})

### Cross-Genre

#### Decisions
- [Audio strategy: minimal music + heavy ambient + voice bites]({path})

## Workflow

### Claude Code
- [Scene attachment check before deleting MonoBehaviour]({path})
- [Backup scene before modify]({path})
- [Context degradation threshold (20-40%)]({path})
- [Console Collapse + unique loop suffix]({path})
- [Stop hook infinite loop risk]({path})
- [/clear vs /compact decision rules]({path})
- [3-level analysis system]({path})
- [Quad-backtick prompt format]({path})
- [DO NOT MOVE hardcoded positions]({path})
- [Intentionally LOW maxCapacity for test racks]({path})
- [Universal cleanup post-migration]({path})
- [Cross-project stack reuse (Eskimo identical)]({path})
- [Iterative checkpoint workflow]({path})

### MCP Tools
- [MCP wildcard permissions format]({path})
- [Skill loading on-demand vs @ reference]({path})

### 3D Models
- [Tripo vocab: firewood is wedge-shaped]({path})
- [Procedural textures in Blender Cycles (commercial)]({path})
- [Blender headless Python generation]({path})
- [ZERO floating / ZERO flickering mandate]({path})
- [Single-material atlas for static props]({path})
- [Tripo (organic) vs Blender MCP (geometric) decision]({path})

### Asset Pipeline
- [Tripo cleanup pipeline]({path})
- [Tripo asymmetric / floating retopo]({path})
- [Audio pipeline: ElevenLabs + Suno + FFmpeg]({path})
- [Polybrush iteration rule (no return to regen)]({path})
- [Polybrush settings for low-poly]({path})
```

Code: when writing this MOC, replace each `{path}` with the actual path from the corresponding entry's `Path:` field. Sort entries within each section alphabetically by title.

---

## 9. Final Report Template

After all batches done, Code writes `<kb-root>\KB_BUILD_REPORT.md`:

```markdown
# KB Build Report

Started: {start_timestamp}
Finished: {end_timestamp}
Duration: {duration}

## Summary
- Total entries processed: 168
- Files created: {actual_count}
- Dedup mergers executed: 5 (see KB_BUILD_PACKAGE.md section 5)
- Flagged entries: {count}
- Skipped entries: {count}

## Files Created (by category)
- engine/lessons: {N}
- engine/patterns: {N}
- engine/anti-patterns: {N}
- genre/tycoon/patterns: {N}
- genre/tycoon/decisions: {N}
- genre/survival/patterns: {N}
- genre/survival/decisions: {N}
- genre/cross-genre/decisions: {N}
- workflow/claude-code: {N}
- workflow/mcp-tools: {N}
- workflow/3d-models: {N}
- workflow/asset-pipeline: {N}

## Dedup Mergers Applied
| Source IDs | Target Path | Notes |
|---|---|---|
| #096, #119 | engine/patterns/storage-rack-family-system.md | merged |
| sweep2-CapsuleCollider, #112 | engine/anti-patterns/script-overrides-prefab-inspector.md | merged |
| #011, #106, #107 | engine/lessons/mesh-exporter-obj-pitfalls.md | merged into 3 subsections |
| sweep2-TreeTypeData, #108 | engine/patterns/scriptable-object-runtime-injection.md | merged |
| #093, #094, #095 | engine/patterns/typography-accessibility-stack.md | merged into 3 sections |

## Flagged Entries
(list any entries where rendering required judgment calls or missing context)

## Skipped Entries  
(list any entries that couldn't be rendered)

## MOC.md
Generated with {N} categorized links.

## Git Commits (this build)
- batch-1-engine-lessons-{sha}
- batch-2-engine-patterns-1-{sha}
- batch-3-engine-patterns-2-{sha}
- batch-4-engine-patterns-3-antipatterns-{sha}
- batch-5-tycoon-patterns-{sha}
- batch-6-tycoon-decisions-survival-{sha}
- batch-7-workflow-claude-code-mcp-{sha}
- batch-8-workflow-3d-pipeline-{sha}

## Next Steps for Hunter
1. Review MOC.md to validate categorization
2. Spot-check 5-10 entries for content quality
3. Identify gaps or wrong categorization, flag for next iteration
4. Use `/kb-review` command in Claude Code to query the KB during development
```

---

## 10. END OF BUILD PACKAGE

This file is complete. Total entries documented: 168 candidates → ~150 unique files after dedup mergers.

**Code's job:**
1. Read this entire document once at start (one /clear cycle pre-build)
2. Generate per-entry .md files using templates + brief data
3. Maintain KB_PROGRESS.md as you go
4. Commit per batch
5. Generate MOC.md at end
6. Generate KB_BUILD_REPORT.md at end

**Estimated time for Code:** 4-8 hours of autonomous work (~2-3 minutes per entry generation, plus commits and validation).

**Hunter does nothing.** This package is autonomous. If Code hits an unresolvable issue, it flags and continues.

🪓 END
