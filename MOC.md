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

Extract these from TT memory (C:\Users\<user>\.claude\projects\D--Unity-Timber-Tycoon\memory\):

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

141 entries created across 8 batches. Links use path relative to `<kb-root>\`.

### Engine

#### Lessons (technical know-how, gotchas)
- [Procedural textures must be baked before FBX export](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/procedural-textures-need-bake.md)
- [bake_space_transform + linked duplicates = 90° rotation bug](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/bake-space-transform-linked-duplicates-rotation-bug.md)
- [FBX export standard settings (Blender → Unity)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/fbx-export-standard-settings-blender-to-unity.md)
- [Asset origin at bottom-center convention](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/asset-origin-bottom-center-convention.md)
- [MeshExporter OBJ pitfalls (3 critical bugs)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/mesh-exporter-obj-pitfalls.md)
- [URP shadow cascade tuning for low-poly terrain](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/urp-shadow-cascade-tuning.md)
- [NEVER save_scene or DestroyImmediate in Play Mode](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/never-destructive-ops-in-play-mode.md)
- [Forward axis = -transform.right (Blender FBX quirk)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/forward-axis-blender-fbx-quirk.md)
- [FreezeAll + automaticInertiaTensor=false zeroes inertia tensor (not restored on unfreeze)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/freeze-inertia-tensor-not-restored.md)
- [Self-collision compound BoxColliders → Physics.IgnoreCollision](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/self-collision-compound-colliders-ignore.md)
- [Minimum turnFactor 0.3 for low-speed arcade steering](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/minimum-turn-factor-arcade-steering.md)
- [Vertex color gamma correction Blender → Unity](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/vertex-color-gamma-correction-blender-to-unity.md)
- [Desaturated colors for low-poly aesthetic](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/desaturated-colors-for-low-poly.md)
- [Separate-objects mapping rule (heightmap limitations)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/separate-objects-mapping-rule.md)
- [Runtime vs Editor script separation](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/runtime-vs-editor-script-separation.md)
- [Editor Scene View input capture](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/editor-scene-view-input-capture.md)
- [CapsuleCollider direction axis cheatsheet](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/capsule-collider-direction-axis.md)
- [Cylindric vs rectangular beams visual contrast](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/cylindric-beams-visual-contrast.md)
- [Read actual code before hypothesizing (debugging discipline)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/read-actual-code-before-hypothesizing.md)
- [Tag assignment: code vs Inspector](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/lessons/tag-assignment-code-vs-inspector.md)

#### Patterns (reusable techniques)
- [Vehicle interaction zones as triggers](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/vehicle-interaction-zones-as-triggers.md)
- [Awake-init for ISaveable with dependencies](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/awake-init-for-isaveable-with-dependencies.md)
- [GetOrAddComponent extension method](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/get-or-add-component-pattern.md)
- [Storage migration: Primary new + Legacy fallback](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/storage-migration-primary-plus-legacy-fallback.md)
- [Before-delete legacy class checklist](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/before-delete-legacy-class-checklist.md)
- [Diegetic 3D button raycast pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/diegetic-3d-button-raycast.md)
- [Camera lock: save → lerp → restore](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/camera-lock-save-lerp-restore.md)
- [Sliding head bandsaw mouse-drag tempo minigame](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/sliding-head-bandsaw-mouse-drag-tempo-minigame.md)
- [MaterialPropertyBlock for runtime color variants](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/material-property-block-runtime-color-variants.md)
- [Quest highlight pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/quest-highlight-pattern.md)
- [River mesh semi-elliptical cross-section](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/river-mesh-semi-ellipse-cross-section.md)
- [4-phase weighted smoothstep day/night transition](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/four-phase-weighted-smoothstep-day-night.md)
- [Procedural skybox sun/moon trick](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/procedural-skybox-sun-moon-trick.md)
- [Custom Editor pattern for generators](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/custom-editor-pattern-for-generators.md)
- [Tool viewmodel as child of camera](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/tool-viewmodel-child-of-camera-pattern.md)
- [AudioManager + Mixer architecture](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/audio-manager-mixer-architecture.md)
- [Parallel architecture pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/parallel-architecture-pattern.md)
- [GameEventSO event channel pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/game-event-so-event-channel.md)
- [ISaveable contract](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/isaveable-contract.md)
- [GameStateMachine pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/game-state-machine-pattern.md)
- [VFX performance budget](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/vfx-performance-budget.md)
- [VFX trigger pattern via GameEventSO](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/vfx-trigger-pattern.md)
- [Mountains hierarchy (front + backdrop)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/mountains-hierarchy-front-and-backdrop.md)
- [Cliff + Waterfall hidden cave](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/cliff-waterfall-hidden-cave.md)
- [Audio Mixer snapshots per game state](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/audio-mixer-snapshots-per-game-state.md)
- [Footstep raycast surface detection](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/footstep-raycast-surface-detection.md)
- [Audio occlusion (raycast + LPF + volume)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/audio-occlusion-lpf-volume.md)
- [AudioReverbZone per environment](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/audio-reverb-zone-per-environment.md)
- [Ambient crossfade zone-based](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/ambient-crossfade-zone-based.md)
- [Typography & accessibility stack](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/typography-accessibility-stack.md)
- [Shared mesh + materials reference](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/shared-mesh-and-materials-reference.md)
- [Rack visual fill alignment](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/rack-visual-fill-alignment.md)
- [Catmull-Rom spline road mesh](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/catmull-rom-spline-road-mesh.md)
- [FlattenTerrainUnderRoad smoothstep](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/flatten-terrain-under-road.md)
- [MeshCollider on roads (stackable)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/mesh-collider-on-roads-stackable.md)
- [ScriptableObject runtime injection](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/scriptable-object-runtime-injection.md)
- [SO propagation chain via parameter passing](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/so-propagation-chain-via-parameters.md)
- [Trunk fall physics config](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/trunk-fall-physics-config.md)
- [VehicleCamera runtime attach/detach](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/vehicle-camera-runtime-attach-detach.md)
- [Vehicle enter/exit choreography](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/vehicle-enter-exit-choreography.md)
- [Universal camera lock (canMove flag)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/universal-camera-lock-canmove-flag.md)
- [StatisticsManager pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/statistics-manager-pattern.md)
- [StorageRackRegistry singleton + auto-register](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/storage-rack-registry-auto-register.md)
- [GLOBAL_ROUTER storage pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/global-router-storage-pattern.md)
- [Dictionary warehouse registry](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/dictionary-warehouse-registry.md)
- [Storage activation gating via upgrade](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/storage-activation-gating-upgrade.md)
- [CrateManager tier progression](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/crate-manager-tier-progression.md)
- [Architectural elements naming convention](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/architectural-naming-convention.md)
- [Collider distribution rule](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/collider-distribution-rule.md)
- [Migration pattern with rollback safety](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/migration-pattern-rollback-safety.md)
- [ReputationLevels data-driven progression](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/reputation-levels-data-driven.md)
- [KioskInteractable + Cube placeholder](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/kiosk-interactable-cube-placeholder.md)
- [ChoppableTree multi-type naming convention](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/choppable-tree-multi-type-naming-convention.md)
- [Single-material atlas for static props](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/single-material-atlas-for-static-props.md)
- [TreeState + StumpState enums](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/patterns/tree-stump-state-machine-enums.md)

#### Anti-Patterns (what NOT to do)
- [Cycles bake for solid colors → black/blurred](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/cycles-bake-for-solid-colors.md)
- [Scene files are binary, never text-edit](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/scene-files-binary-never-edit.md)
- [Legacy code conflict after refactor](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/legacy-code-conflict-after-refactor.md)
- [Rotating Directional Light → black terrain at dawn/dusk](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/rotating-directional-light-day-night.md)
- [Script overrides prefab Inspector values (CapsuleCollider + DirtClump cases)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/script-overrides-prefab-inspector-values.md)
- [Low-poly water side-wave anti-pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/low-poly-water-side-wave.md)
- [Race condition: Start() vs Instantiate parameter](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/race-condition-start-vs-instantiate-parameter.md)
- [bake_space_transform + linked duplicates rotation bug](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/bake-space-transform-linked-duplicates-rotation-bug.md)
- [Generator destroys both paths unconditionally (no per-path guard)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/engine/anti-patterns/generator-destroys-both-paths-no-guard.md)

### Genre

#### Tycoon — Patterns
- [NPC parking PD controller](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/npc-parking-pd-controller.md)
- [Pipeline-style NPC spawn](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/pipeline-style-npc-spawn.md)
- [Initial fill on load](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/initial-fill-on-load.md)
- [NavMesh + kinematic waypoints hybrid](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/navmesh-plus-kinematic-waypoints.md)
- [Object pooling NPCs + FIFO queue](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/object-pooling-npcs-fifo-queue.md)
- [Carry capacity progression + sprint advantage](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/carry-capacity-progression-sprint.md)
- [Multi-step quest checklist](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/multi-step-quest-checklist.md)
- [Customer tier system (Regular/Contractor/VIP)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/customer-tier-system.md)
- [WaterZone gameplay component](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/water-zone-gameplay-component.md)
- [Minecraft-style lighting](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/minecraft-style-lighting.md)
- [ToolWheel UX pattern](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/tool-wheel-ux-pattern.md)
- [WorkerData blueprint + WorkerInstance runtime split](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/worker-data-instance-split.md)
- [Worker SimulateWorkCycle abstraction (no NavMesh)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/worker-simulate-work-cycle.md)
- [Worker output quality distribution](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/worker-output-quality-distribution.md)
- [OrderFulfiller interface (asymmetric player vs NPC)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/order-fulfiller-interface.md)
- [Wing snap-points modular fade-in](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/wing-snap-points-modular-fade-in.md)
- [Debris cleanup single-click drop](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/debris-cleanup-single-click-drop.md)
- [StorageRack family system](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/storage-rack-family-system.md)
- [Visualization ratio (10:1 inventory to visual)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/visualization-ratio.md)
- [Top-down camera minigame (stump digging)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/top-down-minigame-stump-digging.md)
- [Reverse-parking entry stub orientation (forward-tail)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/patterns/reverse-parking-entry-stub-orientation.md)

#### Tycoon — Decisions
- [Quantity-not-quality design principle](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/quantity-not-quality-principle.md)
- [Sales flow: hybrid player + NPC worker](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/sales-flow-decision-hybrid.md)
- [Tool tier replacement (not multiple in inventory)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/tool-tier-replacement-not-inventory.md)
- [VFX wycofane (sawdust/kurz/liście cut from MVP)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/vfx-wycofane-decision.md)
- [Building progression instant spawn (Tavern Manager style)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/building-progression-instant-spawn.md)
- [Player-built vs purchased dichotomy](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/player-built-vs-purchased-dichotomy.md)
- [UI Phase 4 catalog roadmap](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/ui-phase-4-catalog.md)
- [Rack architecture decision (3 options analyzed)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/rack-architecture-decision.md)
- [PlantingSpot universal-not-typed design](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/planting-spot-universal-not-typed.md)
- [Zero code changes per species philosophy](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/zero-code-changes-philosophy.md)
- [LoadingStation: walking vs centralized (QoL upgrade)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/tycoon/decisions/loading-station-decision.md)

#### Survival (Eskimo Simulator)
- [Chunk-based world loading](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/survival/patterns/chunk-based-world-loading.md)
- [Multiplayer-from-MVP (not retrofit)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/survival/decisions/multiplayer-from-mvp-not-retrofit.md)

#### Cross-Genre
- [Audio strategy: minimal music + heavy ambient + voice bites](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/genre/cross-genre/decisions/audio-strategy-minimal-music-heavy-ambient.md)

### Workflow

#### Claude Code
- [Scene attachment check before deleting MonoBehaviour](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/scene-attachment-check-before-deleting.md)
- [Backup scene before modify](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/backup-scene-before-modify.md)
- [Context degradation threshold (20-40%)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/context-degradation-threshold.md)
- [Console Collapse + unique loop suffix](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/console-collapse-loop-suffix.md)
- [Stop hook infinite loop risk](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/stop-hook-infinite-loop-risk.md)
- [/clear vs /compact decision rules](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/clear-vs-compact-decision-rules.md)
- [3-level analysis system](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/three-level-analysis-system.md)
- [Quad-backtick prompt format](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/quad-backtick-prompt-format.md)
- [DO NOT MOVE hardcoded positions](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/do-not-move-hardcoded-positions.md)
- [Intentionally LOW maxCapacity for test racks](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/intentionally-low-maxcapacity-test-racks.md)
- [Universal cleanup post-migration](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/universal-cleanup-post-migration.md)
- [Cross-project stack reuse (Eskimo identical)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/cross-project-stack-reuse.md)
- [Iterative checkpoint workflow](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/claude-code/iterative-checkpoint-workflow.md)

#### MCP Tools
- [MCP wildcard permissions format](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/mcp-tools/mcp-wildcard-permissions-format.md)
- [Skill loading on-demand vs @ reference](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/mcp-tools/skill-loading-on-demand-vs-reference.md)

#### 3D Models
- [Tripo vocab: firewood is wedge-shaped](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/3d-models/tripo-vocab-firewood-wedge.md)
- [Procedural textures in Blender Cycles (commercial)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/3d-models/procedural-textures-cycles-commercial.md)
- [Blender headless Python generation](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/3d-models/blender-headless-python-generation.md)
- [ZERO floating / ZERO flickering mandate](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/3d-models/zero-floating-zero-flickering-mandate.md)
- [Tripo (organic) vs Blender MCP (geometric) — pipeline routing](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/3d-models/tripo-organic-vs-blender-geometric-decision.md)

#### Asset Pipeline
- [Tripo cleanup pipeline](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/asset-pipeline/tripo-cleanup-pipeline.md)
- [Tripo asymmetric / floating retopo](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/asset-pipeline/tripo-asymmetric-floating-retopo.md)
- [Audio pipeline: ElevenLabs + Suno + FFmpeg](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/asset-pipeline/audio-asset-pipeline.md)
- [Polybrush iteration rule (no return to regen)](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/asset-pipeline/polybrush-iteration-rule.md)
- [Polybrush settings for low-poly](https://raw.githubusercontent.com/HunterBright/timber-tycoon-kb/master/workflow/asset-pipeline/polybrush-settings-low-poly.md)
