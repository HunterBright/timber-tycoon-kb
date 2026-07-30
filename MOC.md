---
title: Mapa tresci bazy wiedzy
type: index
status: verified
confidence: high
date: '2026-07-30'
project: GameDevOS
tags:
- indeks
---

# Mapa tresci bazy wiedzy

Wszystkie odnosniki prowadza do notatek w tym vaulcie.
Poprzednia wersja tej mapy kierowala na adresy GitHuba, wiec kazde
klikniecie wyprowadzalo z Obsidiana do przegladarki - patrz `_archive/`.

Legenda statusu: `[OK]` sprawdzone dowodem, `[szkic]` zapisane ale nieprzejrzane,
`[do odtworzenia]` hipoteza wymagajaca reprodukcji, `[zastapione]`, `[odrzucone]`.

## Silnik - lekcje (25)

Bledy rozwiazane i pulapki zlapane w Unity i Blenderze.

- [[urp-distant-caster-shadow-band|"Dark band that follows the player" = terrain self-shadow leaking onto near-coplanar road meshes]] [OK]
- [[bake-space-transform-linked-duplicates-rotation-bug|bake_space_transform + Linked Duplicates = 90° Rotation Injection]] [szkic]
- [[capsule-collider-direction-axis|CapsuleCollider Direction Axis Cheatsheet]] [szkic]
- [[cylindric-beams-visual-contrast|Cylindric vs Rectangular Beams for Visual Contrast]] [szkic]
- [[debugging-search-first-trust-render-check-upstream|Debugging methodology: search-first, trust the render, check for upstream sabotage]] [OK]
- [[desaturated-colors-for-low-poly|Desaturated Colors for Low-Poly Aesthetic]] [szkic]
- [[dynamic-rigidbody-no-nonconvex-meshcollider|Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh]] [szkic]
- [[editor-scene-view-input-capture|Editor Scene View Input Capture Pattern]] [szkic]
- [[fbx-export-standard-settings-blender-to-unity|FBX Export Standard Settings (Blender → Unity)]] [szkic]
- [[forward-axis-blender-fbx-quirk|Forward Axis = -transform.right (Blender FBX Quirk)]] [szkic]
- [[freeze-inertia-tensor-not-restored|FreezeAll + automaticInertiaTensor=false Zeroes the Inertia Tensor]] [szkic]
- [[mesh-exporter-obj-pitfalls|MeshExporter OBJ Pitfalls (3 Critical Bugs)]] [szkic]
- [[minimum-turn-factor-arcade-steering|Minimum turnFactor 0.3 for Low-Speed Arcade Steering]] [szkic]
- [[never-destructive-ops-in-play-mode|NEVER save_scene or DestroyImmediate in Play Mode]] [szkic]
- [[playmode-asset-pollution-vs-disk|Play-Mode in-memory edits pollute on-disk assets — and a "fix" can produce zero git diff]] [OK]
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] [szkic]
- [[runtime-vs-editor-script-separation|Runtime vs Editor Script Separation (Assembly Boundary)]] [szkic]
- [[scene-view-ab-false-positive-game-view-ground-truth|Scene View A/B screenshots gave a false-positive diagnosis — verify in the GAME view with the live camera]] [OK]
- [[scriptableobject-playmode-persistence|ScriptableObject changes in Play Mode DO persist after exit]] [OK]
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] [szkic]
- [[separate-objects-mapping-rule|Separate-Objects Mapping Rule (Heightmap Limitations)]] [szkic]
- [[stale-reflection-probe-night-whitening|Stale Skybox Reflections Whiten PBR Materials at Night (Day/Night Cycle)]] [OK]
- [[tag-assignment-code-vs-inspector|Tag Assignment: Code vs Inspector for Runtime-Spawned Objects]] [szkic]
- [[urp-shadow-cascade-tuning|URP Shadow Cascade Tuning for Low-Poly Terrain]] [szkic]
- [[vertex-color-gamma-correction-blender-to-unity|Vertex Color Gamma Correction Blender → Unity]] [szkic]

## Silnik - wzorce (59)

Sprawdzone sposoby robienia rzeczy.

- [[four-phase-weighted-smoothstep-day-night|4-Phase Weighted Smoothstep Day/Night Transition]] [szkic]
- [[ambient-crossfade-zone-based|Ambient Crossfade Zone-Based Pattern]] [szkic]
- [[architectural-naming-convention|Architectural Elements Naming Convention]] [szkic]
- [[asset-origin-bottom-center-convention|Asset Origin at Bottom-Center Convention]] [szkic]
- [[audio-mixer-snapshots-per-game-state|Audio Mixer Snapshots per Game State]] [szkic]
- [[audio-occlusion-lpf-volume|Audio Occlusion Pattern (LPF + Volume)]] [szkic]
- [[audio-manager-mixer-architecture|AudioManager + Mixer Architecture (5 Channels + 10-Source Pool)]] [szkic]
- [[audio-reverb-zone-per-environment|AudioReverbZone per Environment]] [szkic]
- [[awake-init-for-isaveable-with-dependencies|Awake-Init for ISaveable with Dependencies]] [szkic]
- [[migration-pattern-rollback-safety|Backend Migration Pattern with Rollback Safety]] [szkic]
- [[before-delete-legacy-class-checklist|Before-Delete Legacy Class Checklist]] [szkic]
- [[camera-lock-save-lerp-restore|Camera Lock: Save → Lerp → Restore]] [szkic]
- [[catmull-rom-spline-road-mesh|Catmull-Rom Spline + Quad Strip Mesh for Roads]] [szkic]
- [[choppable-tree-multi-type-naming-convention|ChoppableTree Multi-Type Naming Convention]] [szkic]
- [[cliff-waterfall-hidden-cave|Cliff + Waterfall Hidden Cave Pattern]] [szkic]
- [[collider-distribution-rule|Collider Distribution Rule (Architecture)]] [szkic]
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] [OK]
- [[crate-manager-tier-progression|CrateManager Tier Progression]] [szkic]
- [[custom-editor-pattern-for-generators|Custom Editor Pattern for Generators]] [szkic]
- [[dictionary-warehouse-registry|Dictionary<ProductType, int> Warehouse Registry]] [szkic]
- [[diegetic-3d-button-raycast|Diegetic 3D Button Raycast Pattern]] [szkic]
- [[fbx-long-axis-detect-programmatically|Don't assume an FBX mesh's axis — detect the longest axis programmatically from bounds]] [OK]
- [[flatten-terrain-under-road|Flatten Terrain Under Road (Smoothstep Blend)]] [szkic]
- [[footstep-raycast-surface-detection|Footstep Raycast Surface Detection]] [szkic]
- [[game-event-so-event-channel|GameEventSO ScriptableObject Event Channel]] [szkic]
- [[game-state-machine-pattern|GameStateMachine Pattern]] [szkic]
- [[get-or-add-component-pattern|GetOrAddComponent Extension Method]] [szkic]
- [[global-router-storage-pattern|GLOBAL_ROUTER Pattern (StorageManager.AddToStorage)]] [szkic]
- [[isaveable-contract|ISaveable Contract]] [szkic]
- [[kiosk-interactable-cube-placeholder|KioskInteractable + Cube Placeholder Pattern]] [szkic]
- [[material-property-block-runtime-color-variants|MaterialPropertyBlock for Runtime Color Variants]] [szkic]
- [[mesh-collider-on-roads-stackable|MeshCollider on Roads = Stackable]] [szkic]
- [[mountains-hierarchy-front-and-backdrop|Mountains Hierarchy — Front Ring + Backdrop Double-Sided]] [szkic]
- [[parallel-architecture-pattern|Parallel Architecture Pattern (Locator + Events + ISaveable + Singleton)]] [szkic]
- [[procedural-skybox-sun-moon-trick|Procedural Skybox Sun/Moon Trick]] [szkic]
- [[quest-highlight-pattern|Quest Highlight Pattern (Quest-Flag Mechanism)]] [szkic]
- [[rack-visual-fill-alignment|Rack Visual Fill Alignment Pattern]] [szkic]
- [[reputation-levels-data-driven|ReputationLevels.asset Data-Driven Progression]] [szkic]
- [[river-mesh-semi-ellipse-cross-section|River Mesh Semi-Elliptical Cross-Section]] [szkic]
- [[scriptable-object-runtime-injection|ScriptableObject Runtime Injection Pattern]] [szkic]
- [[shared-mesh-and-materials-reference|Shared Mesh + Materials Reference Pattern]] [szkic]
- [[single-material-atlas-for-static-props|Single-Material Atlas for Static Props]] [szkic]
- [[sliding-head-bandsaw-mouse-drag-tempo-minigame|Sliding Head Bandsaw — Mouse Drag Tempo Minigame]] [szkic]
- [[so-propagation-chain-via-parameters|SO Propagation Chain via Parameter Passing]] [szkic]
- [[stacked-carry-system-camera-viewmodel|Stacked carry system — camera viewmodel + LIFO + species-agnostic prefab refs]] [OK]
- [[statistics-manager-pattern|StatisticsManager Pattern]] [szkic]
- [[storage-activation-gating-upgrade|Storage Activation Gating via Upgrade Purchase]] [szkic]
- [[storage-migration-primary-plus-legacy-fallback|Storage Migration: Primary New + Legacy Fallback]] [szkic]
- [[storage-rack-registry-auto-register|StorageRackRegistry Singleton + Auto-Registration via OnEnable]] [szkic]
- [[tool-viewmodel-child-of-camera-pattern|Tool Viewmodel as Child of Camera]] [szkic]
- [[tree-stump-state-machine-enums|TreeState + StumpState Enums State Machine]] [szkic]
- [[trunk-fall-physics-config|Trunk Fall Physics Config]] [szkic]
- [[typography-accessibility-stack|Typography + Accessibility Stack]] [szkic]
- [[universal-camera-lock-canmove-flag|Universal Camera Lock — canMove Flag]] [szkic]
- [[vehicle-enter-exit-choreography|Vehicle Enter/Exit Choreography Sequence]] [szkic]
- [[vehicle-interaction-zones-as-triggers|Vehicle Interaction Zones as Triggers]] [szkic]
- [[vehicle-camera-runtime-attach-detach|VehicleCamera Third-Person Orbit (Runtime Attach/Detach)]] [szkic]
- [[vfx-performance-budget|VFX Performance Budget]] [szkic]
- [[vfx-trigger-pattern|VFX Trigger Pattern via GameEventSO]] [szkic]

## Silnik - anty-wzorce (10)

Co NIE dziala i dlaczego.

- [[cycles-bake-for-solid-colors|ANTI-PATTERN: Cycles Bake for Solid Color Regions]] [szkic]
- [[generator-destroys-both-paths-no-guard|ANTI-PATTERN: Generator Destroys Both Paths With No Guard]] [szkic]
- [[legacy-code-conflict-after-refactor|ANTI-PATTERN: Legacy Code Conflict After Refactor]] [szkic]
- [[rotating-directional-light-day-night|ANTI-PATTERN: Rotating Directional Light for Day/Night Cycle]] [szkic]
- [[unity-runtime-writes-to-shared-material-asset|Anti-pattern: runtime writes to a shared material ASSET]] [OK]
- [[scene-files-binary-never-edit|ANTI-PATTERN: Scene Files Are Binary — Never Edit as Text]] [szkic]
- [[script-overrides-prefab-inspector-values|ANTI-PATTERN: Script Overrides Prefab Inspector Values]] [szkic]
- [[low-poly-water-side-wave|ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves]] [szkic]
- [[snap-freeze-instead-of-fixing-physics-cause|ANTI-PATTERN: Snap/Freeze to Mask a Physics Bug Instead of Fixing the Cause]] [szkic]
- [[race-condition-start-vs-instantiate-parameter|ANTI-PATTERN: Start() Reads SO Before Parent Sets It]] [szkic]

## Tycoon - wzorce (21)

Wzorce specyficzne dla gatunku.

- [[carry-capacity-progression-sprint|Carry Capacity Progression + Sprint Advantage]] [szkic]
- [[customer-tier-system|Customer Tier System (Regular / Contractor / VIP)]] [szkic]
- [[debris-cleanup-single-click-drop|Debris Cleanup — Single-Click Drop Materials Visual]] [szkic]
- [[initial-fill-on-load|Initial Fill on Load (Don't Serialize NPC State)]] [szkic]
- [[minecraft-style-lighting|Minecraft-Style Lighting (Static Overhead + Decorative Sun)]] [szkic]
- [[multi-step-quest-checklist|Multi-Step Quest Checklist Pattern]] [szkic]
- [[navmesh-plus-kinematic-waypoints|NavMesh + Kinematic Waypoints Hybrid]] [szkic]
- [[npc-parking-pd-controller|NPC Parking PD Controller]] [szkic]
- [[object-pooling-npcs-fifo-queue|Object Pooling for NPCs + FIFO Customer Queue]] [szkic]
- [[order-fulfiller-interface|OrderFulfiller Interface (Player vs NPC, Player Always Faster)]] [szkic]
- [[pipeline-style-npc-spawn|Pipeline-Style NPC Spawn (OnPurchaseComplete Trigger)]] [szkic]
- [[reverse-parking-entry-stub-orientation|Reverse-parking: orientation on the entry stub (forward-tail)]] [szkic]
- [[storage-rack-family-system|StorageRack Family System]] [szkic]
- [[tool-wheel-ux-pattern|ToolWheel UX Pattern]] [szkic]
- [[top-down-minigame-stump-digging|Top-Down Camera Minigame (Stump Digging)]] [szkic]
- [[visualization-ratio|Visualization Ratio (Inventory to Visual Stack)]] [szkic]
- [[water-zone-gameplay-component|WaterZone Gameplay Component]] [szkic]
- [[wing-snap-points-modular-fade-in|Wing Snap-Points Modular Instant Fade-In]] [szkic]
- [[worker-output-quality-distribution|Worker Output Quality Distribution]] [szkic]
- [[worker-simulate-work-cycle|Worker Simulate Work Cycle (No NavMesh/AI)]] [szkic]
- [[worker-data-instance-split|WorkerData (SO Blueprint) + WorkerInstance (Runtime) Split]] [szkic]

## Tycoon - decyzje (12)

Decyzje projektowe Kerf.

- [[building-progression-instant-spawn|Building Progression — Instant Spawn Post-Purchase]] [szkic]
- [[loading-station-decision|LoadingStation Decision — Manual Walk Early, Station Late]] [szkic]
- [[planting-spot-universal-not-typed|PlantingSpot Universal (Not Typed by Species)]] [szkic]
- [[player-built-vs-purchased-dichotomy|Player-Built vs. Purchased Dichotomy]] [szkic]
- [[quantity-not-quality-principle|Quantity-Not-Quality Design Principle]] [szkic]
- [[rack-architecture-decision|Rack Architecture Decision (3 Options)]] [szkic]
- [[sales-flow-decision-hybrid|Sales Flow Decision — Hybrid D (Player + NPC Side by Side)]] [szkic]
- [[tier-system-foundation|Tier System Foundation — Trees, Products, Machines (SOURCE OF TRUTH)]] [szkic]
- [[tool-tier-replacement-not-inventory|Tool Tier Replacement — NOT Multiple Tiers in Inventory]] [szkic]
- [[ui-phase-4-catalog|UI Phase 4 Catalog (12 Systems)]] [szkic]
- [[vfx-wycofane-decision|VFX Wycofane — Sawdust, Kurz, Liście Cut from MVP]] [szkic]
- [[zero-code-changes-philosophy|Zero-Code-Changes Philosophy]] [szkic]

## Survival - wzorce (1)

Pod przyszly projekt.

- [[chunk-based-world-loading|Chunk-Based World Loading (Eskimo Simulator)]] [szkic]

## Survival - decyzje (1)

- [[multiplayer-from-mvp-not-retrofit|Multiplayer from MVP — Not Retrofit]] [szkic]

## Miedzy gatunkami - decyzje (1)

- [[audio-strategy-minimal-music-heavy-ambient|Audio Strategy — Minimal Music + Heavy Ambient + Voice Bites]] [szkic]

## Workflow - jakosc i weryfikacja (2)

Zasady, na ktorych stoi caly system bramek.

- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] [OK]
- [[build-is-the-only-truth-editor-lies|Edytora nie da sie oszukac, zeby udawal build]] [OK]

## Workflow - Claude Code (14)

Agenci, hooki, skille, protokoly.

- [[do-not-move-hardcoded-positions|"DO NOT MOVE" Hardcoded Positions Convention]] [szkic]
- [[clear-vs-compact-decision-rules|/clear vs /compact — Decision Rules]] [szkic]
- [[backup-scene-before-modify|Backup Scene Before Structural Modification]] [szkic]
- [[console-collapse-loop-suffix|Console Collapse + Loop Suffix Convention]] [szkic]
- [[context-degradation-threshold|Context Degradation Threshold (20-40% Remaining)]] [szkic]
- [[cross-project-stack-reuse|Cross-Project Stack Consistency (TT → Eskimo)]] [szkic]
- [[intentionally-low-maxcapacity-test-racks|Intentionally Low maxCapacity for Test Racks]] [szkic]
- [[iterative-checkpoint-workflow|Iterative Checkpoint Workflow for Generated Assets]] [szkic]
- [[quad-backtick-prompt-format|Quad-Backtick Claude Code Prompt Format]] [szkic]
- [[read-actual-code-before-hypothesizing|Read Actual Code Before Hypothesizing]] [szkic]
- [[scene-attachment-check-before-deleting|Scene Attachment Check Before Deleting MonoBehaviour]] [szkic]
- [[stop-hook-infinite-loop-risk|Stop Hook Infinite Loop Risk]] [szkic]
- [[three-level-analysis-system|Three-Level Analysis System]] [szkic]
- [[universal-cleanup-post-migration|Universal Cleanup Post-Migration]] [szkic]

## Workflow - narzedzia MCP (2)

Coplay, Blender, ElevenLabs.

- [[mcp-wildcard-permissions-format|MCP Wildcard Permissions Format]] [szkic]
- [[skill-loading-on-demand-vs-reference|Skill Loading: @Reference vs On-Demand]] [szkic]

## Workflow - pipeline assetow (5)

Tripo, Blender, Unity.

- [[audio-asset-pipeline|Audio Asset Pipeline (ElevenLabs + Suno + FFmpeg)]] [szkic]
- [[polybrush-iteration-rule|Polybrush Iteration Rule — No Return to Generator]] [szkic]
- [[polybrush-settings-low-poly|Polybrush Settings for Low-Poly Terrain]] [szkic]
- [[tripo-asymmetric-floating-retopo|Tripo Asymmetric / Floating Elements — Blender Retopo]] [szkic]
- [[tripo-cleanup-pipeline|Tripo Cleanup Pipeline]] [szkic]

## Workflow - modele 3D (5)

Modelowanie, UV, wypalanie.

- [[blender-headless-python-generation|Blender Headless Python Script Generation]] [szkic]
- [[procedural-textures-cycles-commercial|Procedural Textures in Blender Cycles (Commercial Release Rationale)]] [szkic]
- [[tripo-organic-vs-blender-geometric-decision|Tripo (organic) vs Blender MCP (geometric) — Pipeline Routing Decision]] [szkic]
- [[tripo-vocab-firewood-wedge|Tripo Vocab: Firewood = Wedge-Shaped Piece]] [szkic]
- [[zero-floating-zero-flickering-mandate|ZERO Floating / ZERO Flickering Mandate]] [szkic]

## Projekty (1)

Indeksy projektowe.

- [[timber-tycoon|Timber Tycoon — Project Index]] [OK]

## Pozostale (6)

- [[KB_BUILD_PACKAGE|KB Build Package — Timber Tycoon Knowledge Base]] [szkic] (`_archive`)
- [[MOC-stary-linki-github-2026-07-30|Map of Content — Knowledge Base]] [szkic] (`_archive`)
- [[anti-pattern|<What NOT to do>]] [szkic] (`templates`)
- [[decision|<Decision title>]] [szkic] (`templates`)
- [[lesson|<Title — specific, searchable>]] [szkic] (`templates`)
- [[pattern|<Pattern name>]] [szkic] (`templates`)

## Skrzynka wejsciowa (281)

Drafty czekajace na weryfikacje. Nie sa indeksowane tutaj pojedynczo -
kolejka do przegladu jest w [[PRZEGLAD-INBOXU]].

---

Zaindeksowano 165 wpisow uporzadkowanych. Mapa jest generowana skryptem
`tools/kb_audit.py --write-moc` - nie edytuj jej recznie, bo zmiany przepadna.
