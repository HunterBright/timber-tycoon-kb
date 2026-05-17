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

---

## Generated entries (KB_BUILD_PACKAGE 2026-05-17)

141 entries created across 8 batches. Links use path relative to `D:\Hunter\KnowledgeBase\`.

### Engine

#### Lessons (technical know-how, gotchas)
- [Procedural textures must be baked before FBX export](engine/lessons/procedural-textures-need-bake.md)
- [bake_space_transform + linked duplicates = 90° rotation bug](engine/anti-patterns/bake-space-transform-linked-duplicates-rotation-bug.md)
- [FBX export standard settings (Blender → Unity)](engine/lessons/fbx-export-standard-settings-blender-to-unity.md)
- [Asset origin at bottom-center convention](engine/lessons/asset-origin-bottom-center-convention.md)
- [MeshExporter OBJ pitfalls (3 critical bugs)](engine/lessons/mesh-exporter-obj-pitfalls.md)
- [URP shadow cascade tuning for low-poly terrain](engine/lessons/urp-shadow-cascade-tuning.md)
- [NEVER save_scene or DestroyImmediate in Play Mode](engine/lessons/never-destructive-ops-in-play-mode.md)
- [Forward axis = -transform.right (Blender FBX quirk)](engine/lessons/forward-axis-blender-fbx-quirk.md)
- [Self-collision compound BoxColliders → Physics.IgnoreCollision](engine/lessons/self-collision-compound-colliders-ignore.md)
- [Minimum turnFactor 0.3 for low-speed arcade steering](engine/lessons/minimum-turn-factor-arcade-steering.md)
- [Vertex color gamma correction Blender → Unity](engine/lessons/vertex-color-gamma-correction-blender-to-unity.md)
- [Desaturated colors for low-poly aesthetic](engine/lessons/desaturated-colors-for-low-poly.md)
- [Separate-objects mapping rule (heightmap limitations)](engine/lessons/separate-objects-mapping-rule.md)
- [Runtime vs Editor script separation](engine/lessons/runtime-vs-editor-script-separation.md)
- [Editor Scene View input capture](engine/lessons/editor-scene-view-input-capture.md)
- [CapsuleCollider direction axis cheatsheet](engine/lessons/capsule-collider-direction-axis.md)
- [Cylindric vs rectangular beams visual contrast](engine/lessons/cylindric-beams-visual-contrast.md)
- [Read actual code before hypothesizing (debugging discipline)](engine/patterns/read-actual-code-before-hypothesizing.md)
- [Tag assignment: code vs Inspector](engine/lessons/tag-assignment-code-vs-inspector.md)

#### Patterns (reusable techniques)
- [Vehicle interaction zones as triggers](engine/patterns/vehicle-interaction-zones-as-triggers.md)
- [Awake-init for ISaveable with dependencies](engine/patterns/awake-init-for-isaveable-with-dependencies.md)
- [GetOrAddComponent extension method](engine/patterns/get-or-add-component-pattern.md)
- [Storage migration: Primary new + Legacy fallback](engine/patterns/storage-migration-primary-plus-legacy-fallback.md)
- [Before-delete legacy class checklist](engine/patterns/before-delete-legacy-class-checklist.md)
- [ScriptableObject runtime-assigned references](engine/patterns/scriptable-object-runtime-assigned-references.md)
- [Diegetic 3D button raycast pattern](engine/patterns/diegetic-3d-button-raycast.md)
- [Camera lock: save → lerp → restore](engine/patterns/camera-lock-save-lerp-restore.md)
- [Sliding head bandsaw mouse-drag tempo minigame](engine/patterns/sliding-head-bandsaw-mouse-drag-tempo-minigame.md)
- [MaterialPropertyBlock for runtime color variants](engine/patterns/material-property-block-runtime-color-variants.md)
- [Quest highlight pattern](engine/patterns/quest-highlight-pattern.md)
- [River mesh semi-elliptical cross-section](engine/patterns/river-mesh-semi-ellipse-cross-section.md)
- [4-phase weighted smoothstep day/night transition](engine/patterns/four-phase-weighted-smoothstep-day-night.md)
- [Procedural skybox sun/moon trick](engine/patterns/procedural-skybox-sun-moon-trick.md)
- [Custom Editor pattern for generators](engine/patterns/custom-editor-pattern-for-generators.md)
- [Tool viewmodel as child of camera](engine/patterns/tool-viewmodel-child-of-camera-pattern.md)
- [AudioManager + Mixer architecture](engine/patterns/audio-manager-mixer-architecture.md)
- [Parallel architecture pattern](engine/patterns/parallel-architecture-pattern.md)
- [GameEventSO event channel pattern](engine/patterns/game-event-so-event-channel.md)
- [ISaveable contract](engine/patterns/isaveable-contract.md)
- [GameStateMachine pattern](engine/patterns/game-state-machine-pattern.md)
- [VFX performance budget](engine/patterns/vfx-performance-budget.md)
- [VFX trigger pattern via GameEventSO](engine/patterns/vfx-trigger-pattern.md)
- [Mountains hierarchy (front + backdrop)](engine/patterns/mountains-hierarchy-front-and-backdrop.md)
- [Cliff + Waterfall hidden cave](engine/patterns/cliff-waterfall-hidden-cave.md)
- [Audio Mixer snapshots per game state](engine/patterns/audio-mixer-snapshots-per-game-state.md)
- [Footstep raycast surface detection](engine/patterns/footstep-raycast-surface-detection.md)
- [Audio occlusion (raycast + LPF + volume)](engine/patterns/audio-occlusion-lpf-volume.md)
- [AudioReverbZone per environment](engine/patterns/audio-reverb-zone-per-environment.md)
- [Ambient crossfade zone-based](engine/patterns/ambient-crossfade-zone-based.md)
- [Typography & accessibility stack](engine/patterns/typography-accessibility-stack.md)
- [Shared mesh + materials reference](engine/patterns/shared-mesh-and-materials-reference.md)
- [Rack visual fill alignment](engine/patterns/rack-visual-fill-alignment.md)
- [Catmull-Rom spline road mesh](engine/patterns/catmull-rom-spline-road-mesh.md)
- [FlattenTerrainUnderRoad smoothstep](engine/patterns/flatten-terrain-under-road.md)
- [MeshCollider on roads (stackable)](engine/patterns/mesh-collider-on-roads-stackable.md)
- [ScriptableObject runtime injection](engine/patterns/scriptable-object-runtime-injection.md)
- [SO propagation chain via parameter passing](engine/patterns/so-propagation-chain-via-parameters.md)
- [Trunk fall physics config](engine/patterns/trunk-fall-physics-config.md)
- [VehicleCamera runtime attach/detach](engine/patterns/vehicle-camera-runtime-attach-detach.md)
- [Vehicle enter/exit choreography](engine/patterns/vehicle-enter-exit-choreography.md)
- [Universal camera lock (canMove flag)](engine/patterns/universal-camera-lock-canmove-flag.md)
- [StatisticsManager pattern](engine/patterns/statistics-manager-pattern.md)
- [StorageRackRegistry singleton + auto-register](engine/patterns/storage-rack-registry-auto-register.md)
- [GLOBAL_ROUTER storage pattern](engine/patterns/global-router-storage-pattern.md)
- [Dictionary warehouse registry](engine/patterns/dictionary-warehouse-registry.md)
- [Storage activation gating via upgrade](engine/patterns/storage-activation-gating-upgrade.md)
- [CrateManager tier progression](engine/patterns/crate-manager-tier-progression.md)
- [Architectural elements naming convention](engine/patterns/architectural-naming-convention.md)
- [Collider distribution rule](engine/patterns/collider-distribution-rule.md)
- [Migration pattern with rollback safety](engine/patterns/migration-pattern-rollback-safety.md)
- [ReputationLevels data-driven progression](engine/patterns/reputation-levels-data-driven.md)
- [KioskInteractable + Cube placeholder](engine/patterns/kiosk-interactable-cube-placeholder.md)
- [ChoppableTree multi-type naming convention](engine/patterns/choppable-tree-multi-type-naming-convention.md)
- [Single-material atlas for static props](engine/patterns/single-material-atlas-for-static-props.md)
- [TreeState + StumpState enums](engine/patterns/tree-stump-state-machine-enums.md)

#### Anti-Patterns (what NOT to do)
- [Cycles bake for solid colors → black/blurred](engine/anti-patterns/cycles-bake-for-solid-colors.md)
- [Scene files are binary, never text-edit](engine/anti-patterns/scene-files-binary-never-edit.md)
- [Legacy code conflict after refactor](engine/anti-patterns/legacy-code-conflict-after-refactor.md)
- [Rotating Directional Light → black terrain at dawn/dusk](engine/anti-patterns/rotating-directional-light-day-night.md)
- [Script overrides prefab Inspector values (CapsuleCollider)](engine/anti-patterns/script-overrides-prefab-collider.md)
- [Script overrides prefab inspector values (original)](engine/anti-patterns/script-overrides-prefab-inspector-values.md)
- [Low-poly water side-wave anti-pattern](engine/anti-patterns/low-poly-water-side-wave.md)
- [Race condition: Start() vs Instantiate parameter](engine/anti-patterns/race-condition-start-vs-instantiate-parameter.md)
- [bake_space_transform + linked duplicates rotation bug](engine/anti-patterns/bake-space-transform-linked-duplicates-rotation-bug.md)

### Genre

#### Tycoon — Patterns
- [NPC parking PD controller](genre/tycoon/patterns/npc-parking-pd-controller.md)
- [Pipeline-style NPC spawn](genre/tycoon/patterns/pipeline-style-npc-spawn.md)
- [Initial fill on load](genre/tycoon/patterns/initial-fill-on-load.md)
- [NavMesh + kinematic waypoints hybrid](genre/tycoon/patterns/navmesh-plus-kinematic-waypoints.md)
- [Object pooling NPCs + FIFO queue](genre/tycoon/patterns/object-pooling-npcs-fifo-queue.md)
- [Carry capacity progression + sprint advantage](genre/tycoon/patterns/carry-capacity-progression-sprint.md)
- [Multi-step quest checklist](genre/tycoon/patterns/multi-step-quest-checklist.md)
- [Customer tier system (Regular/Contractor/VIP)](genre/tycoon/patterns/customer-tier-system.md)
- [WaterZone gameplay component](genre/tycoon/patterns/water-zone-gameplay-component.md)
- [Minecraft-style lighting](genre/tycoon/patterns/minecraft-style-lighting.md)
- [ToolWheel UX pattern](genre/tycoon/patterns/tool-wheel-ux-pattern.md)
- [WorkerData blueprint + WorkerInstance runtime split](genre/tycoon/patterns/worker-data-instance-split.md)
- [Worker SimulateWorkCycle abstraction (no NavMesh)](genre/tycoon/patterns/worker-simulate-work-cycle.md)
- [Worker output quality distribution](genre/tycoon/patterns/worker-output-quality-distribution.md)
- [OrderFulfiller interface (asymmetric player vs NPC)](genre/tycoon/patterns/order-fulfiller-interface.md)
- [Wing snap-points modular fade-in](genre/tycoon/patterns/wing-snap-points-modular-fade-in.md)
- [Debris cleanup single-click drop](genre/tycoon/patterns/debris-cleanup-single-click-drop.md)
- [StorageRack family system](genre/tycoon/patterns/storage-rack-family-system.md)
- [Visualization ratio (10:1 inventory to visual)](genre/tycoon/patterns/visualization-ratio.md)
- [Top-down camera minigame (stump digging)](genre/tycoon/patterns/top-down-minigame-stump-digging.md)

#### Tycoon — Decisions
- [Quantity-not-quality design principle](genre/tycoon/decisions/quantity-not-quality-principle.md)
- [Sales flow: hybrid player + NPC worker](genre/tycoon/decisions/sales-flow-decision-hybrid.md)
- [Tool tier replacement (not multiple in inventory)](genre/tycoon/decisions/tool-tier-replacement-not-inventory.md)
- [VFX wycofane (sawdust/kurz/liście cut from MVP)](genre/tycoon/decisions/vfx-wycofane-decision.md)
- [Building progression instant spawn (Tavern Manager style)](genre/tycoon/decisions/building-progression-instant-spawn.md)
- [Player-built vs purchased dichotomy](genre/tycoon/decisions/player-built-vs-purchased-dichotomy.md)
- [UI Phase 4 catalog roadmap](genre/tycoon/decisions/ui-phase-4-catalog.md)
- [Rack architecture decision (3 options analyzed)](genre/tycoon/decisions/rack-architecture-decision.md)
- [PlantingSpot universal-not-typed design](genre/tycoon/decisions/planting-spot-universal-not-typed.md)
- [Zero code changes per species philosophy](genre/tycoon/decisions/zero-code-changes-philosophy.md)
- [LoadingStation: walking vs centralized (QoL upgrade)](genre/tycoon/decisions/loading-station-decision.md)

#### Survival (Eskimo Simulator)
- [Chunk-based world loading](genre/survival/patterns/chunk-based-world-loading.md)
- [Multiplayer-from-MVP (not retrofit)](genre/survival/decisions/multiplayer-from-mvp-not-retrofit.md)

#### Cross-Genre
- [Audio strategy: minimal music + heavy ambient + voice bites](genre/cross-genre/decisions/audio-strategy-minimal-music-heavy-ambient.md)

### Workflow

#### Claude Code
- [Scene attachment check before deleting MonoBehaviour](workflow/claude-code/scene-attachment-check-before-deleting.md)
- [Backup scene before modify](workflow/claude-code/backup-scene-before-modify.md)
- [Context degradation threshold (20-40%)](workflow/claude-code/context-degradation-threshold.md)
- [Console Collapse + unique loop suffix](workflow/claude-code/console-collapse-loop-suffix.md)
- [Stop hook infinite loop risk](workflow/claude-code/stop-hook-infinite-loop-risk.md)
- [/clear vs /compact decision rules](workflow/claude-code/clear-vs-compact-decision-rules.md)
- [3-level analysis system](workflow/claude-code/three-level-analysis-system.md)
- [Quad-backtick prompt format](workflow/claude-code/quad-backtick-prompt-format.md)
- [DO NOT MOVE hardcoded positions](workflow/claude-code/do-not-move-hardcoded-positions.md)
- [Intentionally LOW maxCapacity for test racks](workflow/claude-code/intentionally-low-maxcapacity-test-racks.md)
- [Universal cleanup post-migration](workflow/claude-code/universal-cleanup-post-migration.md)
- [Cross-project stack reuse (Eskimo identical)](workflow/claude-code/cross-project-stack-reuse.md)
- [Iterative checkpoint workflow](workflow/claude-code/iterative-checkpoint-workflow.md)

#### MCP Tools
- [MCP wildcard permissions format](workflow/mcp-tools/mcp-wildcard-permissions-format.md)
- [Skill loading on-demand vs @ reference](workflow/mcp-tools/skill-loading-on-demand-vs-reference.md)

#### 3D Models
- [Tripo vocab: firewood is wedge-shaped](workflow/3d-models/tripo-vocab-firewood-wedge.md)
- [Procedural textures in Blender Cycles (commercial)](workflow/3d-models/procedural-textures-cycles-commercial.md)
- [Blender headless Python generation](workflow/3d-models/blender-headless-python-generation.md)
- [ZERO floating / ZERO flickering mandate](workflow/3d-models/zero-floating-zero-flickering-mandate.md)

#### Asset Pipeline
- [Tripo cleanup pipeline](workflow/asset-pipeline/tripo-cleanup-pipeline.md)
- [Tripo asymmetric / floating retopo](workflow/asset-pipeline/tripo-asymmetric-floating-retopo.md)
- [Audio pipeline: ElevenLabs + Suno + FFmpeg](workflow/asset-pipeline/audio-asset-pipeline.md)
- [Polybrush iteration rule (no return to regen)](workflow/asset-pipeline/polybrush-iteration-rule.md)
- [Polybrush settings for low-poly](workflow/asset-pipeline/polybrush-settings-low-poly.md)
