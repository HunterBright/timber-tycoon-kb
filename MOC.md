# Map of Content — Knowledge Base

Entry point. Browse by category or use full-text search.

## Categories

### Engine (Unity/Blender/URP)
- engine/lessons/ — bugs solved, gotchas captured
- engine/patterns/ — validated workflows
- engine/decisions/ — ADRs with trade-offs
- engine/anti-patterns/ — what NOT to do, with reasons

### Genre
- genre/tycoon/ — Timber Tycoon learnings
- genre/survival/ — future project
- genre/roguelike/ — future project
- genre/pvp-multiplayer/ — future project

### Workflow
- workflow/claude-code/ — agents, hooks, skills, prompts, KB protocol
- workflow/mcp-tools/ — Coplay, Blender, ElevenLabs MCP
- workflow/asset-pipeline/ — Tripo → Blender → Unity
- workflow/3d-models/ — modeling, UV, bake conventions

### Projects (indexes only, no content)
- projects/timber-tycoon.md

## TODO — Seed extraction (post-demo)

Extract these from TT memory (C:\Users\mrhun\.claude\projects\D--Unity-Timber-Tycoon\memory\):

**To engine/lessons/:**
- feedback_bake_space_transform_empties.md → Blender empty + shared mesh rotation bug
- feedback_fbx_importer_remap_unreliable.md → Unity FBX material remap workaround
- feedback_one_mat_per_mesh.md → Material slot reordering rule
- feedback_blender_no_factory_reset.md → Blender API gotcha
- feedback_blender_material_preview.md → Post-task EEVEE restore pattern

**To workflow/asset-pipeline/:**
- feedback_interactive_material_workflow.md → bmesh selection workflow
- feedback_blender_tripo_editing.md → Tripo model editing rules
- feedback_firewood_reference.md → UV/geometry template pattern

**To engine/patterns/ (extract from tt-CLAUDE.md):**
- Diegetic 3D button raycast
- Camera lock pattern for minigames

**To engine/anti-patterns/ (extract from tt-CLAUDE.md):**
- Legacy code conflict after refactor
- Smart UV Project + Cycles bake for solid colors

## Inbox status
See _inbox/ for pending drafts.
