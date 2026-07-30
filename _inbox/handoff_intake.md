---
title: Handoff Intake — 2026-05-17
type: note
status: draft
confidence: low
verified: ''
date: ''
project: Kerf - Sawmill Tycoon
tags: []
applies_to: []
source: ''
---

# Handoff Intake — 2026-05-17

## Section 1: KB folder tree

```
Folder PATH listing
Volume serial number is 5CE1-CD09
D:\HUNTER\KNOWLEDGEBASE
|   .gitignore
|   CLAUDE.md
|   MOC.md
|   README.md
|   
+---engine
|   +---anti-patterns
|   |       .gitkeep
|   |       
|   +---decisions
|   |       .gitkeep
|   |       
|   +---lessons
|   |       .gitkeep
|   |       
|   \---patterns
|           .gitkeep
|           
+---genre
|   +---pvp-multiplayer
|   |   +---decisions
|   |   |       .gitkeep
|   |   |       
|   |   +---lessons
|   |   |       .gitkeep
|   |   |       
|   |   \---patterns
|   |           .gitkeep
|   |           
|   +---roguelike
|   |   +---decisions
|   |   |       .gitkeep
|   |   |       
|   |   +---lessons
|   |   |       .gitkeep
|   |   |       
|   |   \---patterns
|   |           .gitkeep
|   |           
|   +---survival
|   |   +---decisions
|   |   |       .gitkeep
|   |   |       
|   |   +---lessons
|   |   |       .gitkeep
|   |   |       
|   |   \---patterns
|   |           .gitkeep
|   |           
|   \---tycoon
|       +---decisions
|       |       .gitkeep
|       |       
|       +---lessons
|       |       .gitkeep
|       |       
|       \---patterns
|               .gitkeep
|               
+---projects
|       timber-tycoon.md
|       
+---templates
|       anti-pattern.md
|       decision.md
|       lesson.md
|       pattern.md
|       
+---workflow
|   +---3d-models
|   |       .gitkeep
|   |       
|   +---asset-pipeline
|   |       .gitkeep
|   |       
|   +---claude-code
|   |       .gitkeep
|   |       
|   \---mcp-tools
|           .gitkeep
|           
+---_archive
|       .gitkeep
|       
\---_inbox
        README.md
```

## Section 2: Templates

### D:\Hunter\KnowledgeBase\templates\lesson.md

```md
---
type: lesson
project: <project-name>
suggested-category: engine/lessons
tags: [tag1, tag2]
severity: medium
time_lost: ""
date: YYYY-MM-DD
status: draft
applies_to: []
---

# <Title — specific, searchable>

## Problem
What happened. Concrete symptoms.

## Root cause
The underlying mechanism. Why it happened.

## Solution
What worked. Step by step if applicable.

## What didn't work
Anti-patterns tried during diagnosis.

## Transferability
Why this applies beyond current project.

## Related
- Links to other KB entries
```

### D:\Hunter\KnowledgeBase\templates\pattern.md

```md
---
type: pattern
project: <project-name>
suggested-category: workflow/asset-pipeline
tags: []
date: YYYY-MM-DD
status: draft
---

# <Pattern name>

## When to use
Triggers / conditions where this pattern applies.

## Steps
Concrete sequence.

## Why this works
Underlying mechanism.

## Trade-offs
What you give up by choosing this.

## Variants
Alternative versions for different contexts.
```

### D:\Hunter\KnowledgeBase\templates\decision.md

```md
---
type: decision
project: <project-name>
suggested-category: engine/decisions
tags: []
date: YYYY-MM-DD
status: draft
supersedes: ""
---

# <Decision title>

## Context
What situation forced the decision.

## Options considered
- A: pros / cons
- B: pros / cons
- C: pros / cons

## Chosen
A, because <reasoning>.

## Consequences
What this commits us to.

## Revisit when
Conditions under which this should be reconsidered.
```

### D:\Hunter\KnowledgeBase\templates\anti-pattern.md

```md
---
type: anti-pattern
project: <project-name>
suggested-category: engine/anti-patterns
tags: []
date: YYYY-MM-DD
status: draft
---

# <What NOT to do>

## The trap
What seems like a good idea but isn't.

## Why it fails
Mechanism of failure.

## Symptoms
How you know you're in this trap.

## Correct approach
What to do instead. Link to pattern if applicable.
```

## Section 3: Existing KB entries

### CLAUDE.md — 1043 bytes

```md
# KB-local Claude Code instructions

When Claude Code reads files inside this knowledge base, treat them as authoritative reference:
- Lessons and patterns here are validated by Hunter
- If a project-level CLAUDE.md conflicts with a KB entry, project wins (project is authoritative for its own state)
- KB entries describe transferable wisdom, not current project state

## Status field semantics
- draft — in _inbox/, not yet reviewed
- validated — Hunter-approved, safe to follow
- superseded — newer entry exists, see "supersedes:" field
- deprecated — no longer valid (e.g. Unity API change)

## Entry naming
- Lowercase, hyphen-separated, descriptive
- Format: <topic>-<specific>.md
- Example: urp-multi-material-bake-bug.md, NOT unity-bug-1.md

## KB vs project memory
KB is transferable cross-project knowledge. Project memory (~/.claude/projects/<project>/memory/) is per-project, machine-local state. Don't confuse them. If unsure where something belongs, default to project memory and let Hunter promote to KB during review.
```

### MOC.md — 1931 bytes

```md
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
```

### README.md — 1716 bytes

```md
# Hunter's Knowledge Base

Long-term, cross-project knowledge base for Unity solo dev work.
Survives projects. Grows with each session.

## What lives here vs. what lives in memory

**KB (this folder)** — transferable cross-project wisdom:
- Unity/Blender bugs and gotchas (engine-level)
- Validated workflow patterns (asset pipeline, Claude Code conventions)
- Architecture Decision Records with trade-offs
- Anti-patterns: what NOT to do

**Project memory** (lives at C:\Users\<user>\.claude\projects\<project>\memory\):
- Project-specific feedback (colors, positions, design preferences)
- Current sprint state
- Per-project decisions still in flux
- Machine-local, not in git

Different scope, different lifetime, different storage. Both intentional.

## Structure
- engine/ — Unity/Blender/URP wisdom (transferable across all projects)
- genre/ — Genre-specific patterns (tycoon, survival, roguelike, pvp)
- workflow/ — Meta-tooling: Claude Code, MCP, asset pipeline
- projects/ — Per-project indexes (links only, no content)
- _inbox/ — Drafts awaiting weekly review (Claude writes here)
- _archive/ — Superseded entries (kept for history)
- templates/ — Entry templates (lesson/pattern/decision/anti-pattern)

## Workflow
1. During sessions, Claude Code auto-writes drafts to _inbox/
2. Sunday evening: /kb-review — batch approve/edit/reject drafts
3. New projects: junction kb\ → this folder, instant access

## How Claude Code uses this
Global CLAUDE.md at C:\Users\<user>\.claude\CLAUDE.md instructs Claude to:
- Read MOC.md when task is non-trivial and likely has prior art
- Auto-write drafts when triggers fire (see global CLAUDE.md)
- Not interrupt flow with "should I save?" prompts
```

### projects\timber-tycoon.md — 559 bytes

```md
---
type: project-index
project: timber-tycoon
status: active
demo: pending
---

# Timber Tycoon — Project Index

Single-player FPP tycoon, Unity 6000.3.5f1 / URP 17.3.

## Active KB entries
(Empty — seed pending post-demo extraction)

## TT-specific memory
Lives outside KB at:
C:\Users\<user>\.claude\projects\D--Unity-Timber-Tycoon\memory\

Reason: project-specific feedback (colors, positions, design decisions) is not transferable and stays in user-scoped Claude Code memory.

## Extraction TODO (post-demo)
See MOC.md → "TODO — Seed extraction".
```

### _inbox\README.md — 613 bytes

```md
# Inbox — Pending drafts

Claude Code writes draft entries here during sessions. Hunter reviews weekly.

## Filename format
YYYYMMDD-HHMM-<topic-slug>.md
Example: 20260517-1430-urp-shadow-cascade-fix.md

## Review workflow
Run /kb-review or say "review KB inbox":
1. Claude lists all drafts with title + suggested category + 2-line summary
2. Per draft, Hunter decides: approve / edit / move category / reject / postpone
3. Claude applies decisions, moves files to target folders

## Don't manually move files from here
Let Claude handle the move during review — it preserves git history and updates indexes.
```

engine/ subfolders (anti-patterns/, decisions/, lessons/, patterns/): no .md files — only .gitkeep
genre/ subfolders (tycoon/, survival/, roguelike/, pvp-multiplayer/ each with decisions/, lessons/, patterns/): no .md files — only .gitkeep
workflow/ subfolders (3d-models/, asset-pipeline/, claude-code/, mcp-tools/): no .md files — only .gitkeep
_archive/: no .md files — only .gitkeep

## Section 4: Global CLAUDE.md

```md
# Global Claude Code instructions — Hunter

Loaded for ALL Hunter's projects. Project-level CLAUDE.md (e.g. D:\Unity\Timber_Tycoon\CLAUDE.md) takes precedence on conflicts.

## Hunter context
Creative director, non-programmer. Works in Unity (currently 6000.3.5f1 / URP 17.3). Delegates implementation to Claude Code. Polish/English bilingual.

## Knowledge Base protocol

KB lives at D:\Hunter\KnowledgeBase\. In projects, mounted as junction at <project-root>\kb\ (NOT inside .claude\ — kept separate from per-project git-tracked config).

### Reading the KB
At start of sessions in projects with kb\ junction:
1. Don't read everything — would be context bloat
2. Identify task domain: Unity/Blender? Asset pipeline? Genre-specific?
3. Read kb\MOC.md only if task is non-trivial AND likely to have prior art
4. Read specific entries on demand based on task keywords

### Writing drafts to kb\_inbox\

**Triggers — write a draft when:**
1. A bug took >30 min to diagnose AND root cause is transferable (engine-level, not project-specific)
2. Hunter says "this is the same problem as before" / "we already solved this" — meta-signal that knowledge wasn't captured
3. A workflow was validated and could repeat in future projects
4. A design decision was made with explicit trade-offs (good ADR candidate)
5. An anti-pattern was discovered: "we tried X, doesn't work because Y"

**Anti-triggers — do NOT write a draft for:**
- Project-specific state (colors, positions, asset names, design preferences) → these go to project memory instead
- Trivial fixes (typos, missing imports, path errors)
- Pure creative decisions without technical reasoning
- Work-in-progress not yet validated by Hunter

**Decision rule:** "If this knowledge would help on a future Unity project of different genre, it's KB material. If only relevant to current game's design, it's project memory."

### Draft filename and format

Filename: kb\_inbox\YYYYMMDD-HHMM-<topic-slug>.md (use current timestamp + descriptive slug)

Use templates from kb\templates\ — pick lesson / pattern / decision / anti-pattern based on content type.

Front-matter required:
- type: lesson | pattern | decision | anti-pattern
- project: current project name
- suggested-category: target folder (e.g. engine/lessons)
- tags: array of keywords
- date: today
- status: draft

### Don't ask, just write
Auto-write drafts during the session. Don't interrupt Hunter's flow to ask "should I save this?". Hunter reviews drafts batch-style weekly.

### End-of-session signal
If you wrote drafts during the session, at the end include a brief signal:

> KB drafts written this session:
> - 20260517-1430-urp-shadow-cascade-fix.md (lesson)
> - 20260517-1612-claude-code-3-level-analysis.md (pattern)

Don't summarize content. Just signal existence.

## /kb-review command

When Hunter says "review KB inbox", "/kb-review", or "przejrzyj inbox":
1. List all files in kb\_inbox\ (newest first)
2. For each: show title, suggested category, severity (if lesson), 2-line summary
3. Ask per draft: approve (move to suggested category) / edit / move to different category / reject (delete) / postpone (leave in inbox)
4. Apply decisions in batch
5. If approving, change status: draft → status: validated in front-matter
6. Report final state: N approved, N edited, N rejected, N postponed

## KB vs project memory distinction

Claude Code stores project memory at C:\Users\<user>\.claude\projects\<sanitized-project-path>\memory\. This is per-project, machine-local, NOT git-tracked.

Different from KB:
- Project memory = current project state, feedback, in-flux decisions
- KB = transferable wisdom across projects

When in doubt where something belongs: project memory by default, promote to KB during weekly review if it proves transferable.

## Important
- Project CLAUDE.md takes precedence on conflicts
- Don't modify validated KB entries without explicit Hunter approval
- Archive (don't delete) superseded entries — move to _archive\ with status: superseded and add supersedes: field pointing to new entry
```

## Section 5: Project-level CLAUDE.md

Path checked: D:\Unity\Timber_Tycoon\CLAUDE.md — EXISTS

```md
# Timber Tycoon — Kontekst Projektu

## Przegląd
Single-player FPP tycoon o tartaku. Gracz wycina drzewa, przetwarza drewno, sprzedaje produkty, rozbudowuje tartak.
- **Unity 6000.3.5f1**, URP 17.3, New Input System 1.17
- **Pipeline**: Hunter (reżyser/decyzje) → Claude Code (implementacja) → MCP (Coplay, Blender, ElevenLabs)
- **Plan**: `Docs/TIMBER_TYCOON_FINAL_PLAN.md` — jedyne źródło prawdy o scope
- **Postęp**: @.claude/skills/progress-tracker/SKILL.md
- **Mapa**: @.claude/skills/map-environment/SKILL.md
- **Modele 3D**: @.claude/skills/3d-models-assets/SKILL.md
- **Unity API**: ładowany jako plugin — `claude --plugin-dir ./unity-claude-skills`

## Hunter — kontekst użytkownika

Hunter nie programuje i nie tworzy gier technicznie. Jest creative directorem — decyduje CO ma być zrobione, WIZUALNIE i FUNKCJONALNIE. Implementację technicznie deleguje do Claude Code.

Konsekwencje:
- Wyjaśniaj łopatologicznie — bez żargonu programistycznego, chyba że nazwiesz system zaimplementowany w projekcie (`ServiceLocator`, `GameEventSO`)
- Nie zakładaj, że Hunter rozpozna bug w kodzie — TY masz go znaleźć i opisać
- Przy propozycjach technicznych — zawsze tłumacz DLACZEGO, nie tylko CO
- Hunter ocenia wyniki na podstawie efektu wizualnego/gameplay, nie elegancji kodu

Pełny protokół współpracy: `.claude/skills/hunter-communication-style.md`

---

## Workflow obowiązkowy — 3 poziomy analizy przed implementacją

**Każde zadanie od Huntera klasyfikuj do jednego z 3 poziomów PRZED rozpoczęciem pracy. Jeśli niepewny, który poziom pasuje → zawsze wybierz wyższy (bezpieczniejszy).**

### Poziom 1 — PEŁNA ANALIZA + AKCEPT (obowiązkowa)

**Kiedy stosuj:**
- Refaktor istniejącego systemu
- Zmiany dotykające 3+ plików
- Zmiany architektoniczne (nowy wzorzec, nowy manager, zmiana istniejącego wzorca)
- Dotknięcie hardcoded constraints: pozycja tartaku (177.9, 7.62, -88.71), klif, wodospad, performance budget, forward axis auta (-transform.right), scale inconsistency Spruce_Log/Birch_Log
- Zmiany dotykające zapisu (ISaveable) — migracja save'ów
- Zadania w Plan Mode (Shift+Tab)
- Pytania informacyjne: "co myślisz", "jak byś to zrobił", "co polecasz", "jak najlepiej", "mam dylemat", "nie wiem czy"

**Co robisz:**
1. **Analiza** (co widzisz jako problem/zadanie, jaki jest kontekst w grze, jakie systemy się łączą)
2. **Opcje** (2-3 możliwe rozwiązania z pros/cons każdego w kontekście Timber Tycoon)
3. **Rekomendacja** (które rozwiązanie wybierasz i dlaczego — połącz z istniejącą architekturą)
4. **Pytanie akceptacyjne** ("Czy akceptujesz rozwiązanie X, czy chcesz żebym wrócił do Y/Z?")
5. **STOP** — czekaj na akcept Huntera. NIE implementuj.
6. Jeśli Hunter zgłosi uwagi → ponów krok 1-4 z uwzględnieniem jego uwag, przedstaw za/przeciw jego sugestii, zarekomenduj ulepszone rozwiązanie.
7. Dopiero po "ok/tak/zatwierdzam/rób" → implementuj.

### Poziom 2 — KRÓTKIE UZASADNIENIE + AKCEPT (1-2 zdania)

**Kiedy stosuj:**
- Nowy feature addytywny, który pasuje do istniejącego wzorca (np. nowy typ drzewa — pełni wzór Spruce/Birch/Oak/Maple)
- Nowy plik kodu zgodny z architekturą (ISaveable + GameEventSO + ServiceLocator)
- Zmiana wielu parametrów jednego ScriptableObject
- Nowy prefab według ustalonego pipeline (model → bake → FBX → Unity)

**Co robisz:**
1. **Krótkie uzasadnienie** (1-2 zdania: dlaczego to rozwiązanie, jak pasuje do projektu)
2. **Pytanie akceptacyjne** ("Potwierdzasz?")
3. Po akcept → implementuj.

### Poziom 3 — OD RAZU IMPLEMENTACJA (bez analizy)

**Kiedy stosuj:**
- Zmiana jednej wartości w istniejącym SO (color, speed, capacity, amount)
- Przemianowanie zmiennej/pliku
- Usuwanie pojedynczego komponentu z obiektu
- Literówki, formatowanie, komentarze w kodzie
- Przesunięcie obiektu na scenie o zadaną wartość (bez zmiany architektury)

**Co robisz:**
Implementuj od razu. Po zakończeniu — krótki raport co zostało zmienione.

---

## Zasady — BEZWZGLĘDNE (nigdy nie naruszaj niezależnie od poziomu)

- **ZAWSZE backup sceny** PRZED modyfikacjami scen
- **NIGDY save_scene w Play Mode** — niszczy obiekty permanentnie
- **NIGDY editorowe skrypty w Play Mode** — DestroyImmediate = utrata danych
- **NIGDY nie implementuj nic spoza planu** (`Docs/TIMBER_TYCOON_FINAL_PLAN.md`) bez potwierdzenia Huntera
- **Gameplay values ZAWSZE w ScriptableObjects** — zero magic numbers w kodzie
- **Performance**: max 2M verts, 500 draw calls (target: GTX 1050 / RX 560+, 60 FPS minimum)
- **Git**: conventional commits, LFS dla binarnych

---

## Kluczowe systemy

### Drzewa — multi-type via ScriptableObject
- **TreeTypeData** (SO w Core/): definiuje typ — prefaby (adult, stump, trunk, log, sapling), modele wzrostu
- Gotowe: **Spruce**, **Birch**, **Oak**, **Maple**. Planowane: Acacia, Mahogany
- Cykl: ścięcie → pniak + kłody + sadzonka → wykopanie pniaka → PlantingSpot → sadzenie → GrowingTree (4 etapy) → dorosłe
- Sadzonka niesie `treeTypeData` — PlantingSpot uniwersalny, typ zależy od sadzonki
- **Kłody NIGDY nie są sprzedawane** — zawsze przetwarzane na produkty

### Magazyn — StorageRack system (2026-04-10 refactor)
- **StorageRack** MonoBehaviour per instancja z UUID, **StorageRackRegistry** singleton, **StorageManager** fasada
- Każdy rack = osobna pula, mix (ProductType × WoodSpecies). Pojemności: LogRack=60, FirewoodRack=100, StumpRack=15
- 10 gatunków w WoodSpecies enum (4 aktywne: Spruce/Birch/Oak/Maple)
- **UnloadZone**: trigger przy rampie, klawisz **F** z poziomu auta
- **CarryCrate v2**: mix crate (Tavern Manager + Supermarket Sim), 3 FBX (10/15/25 slotów), koło Q, RackTransferUI, SalesCounterTransferUI

### Pojazdy
- Forward auta = **-transform.right** (quirk eksportu FBX z Blendera)
- **VehicleStorage**: ISaveable, generyczny CargoItem, LIFO, wizualizacja prefabami
- **TruckManager**: cargo po (ProductType, WoodSpecies), 3 TruckData SO (24/40/60)

### Sprzedaż
- Tavern Manager style — NPC przychodzi, gracz nosi produkty w skrzynce do lady
- **CustomerManager**: kolejka zamówień, per species matching
- **EconomyManager**: pieniądze, ISaveable
- **WorkerManager**: NPC pracownicy, 3 niezależne lady, per-rola dictionary

### Processing Chain
- **MachineController** + **MachineOutputCalculator** z ProductType + WoodSpecies propagation
- **ChoppingBlock**: minigierka SwingArc, output BasicFirewood (species-agnostic)
- PlankMaker multi-cycle, Chipper mnożniki, FurnitureWorkshop recepty per typ

## Struktura kodu
```
Assets/Project/Scripts/
├── Core/          — ServiceLocator, GameEventSO, GameState, ISaveable, TreeTypeData, Interactable, QuestManager
├── Inventory/     — InventorySystem (rejestr magazynowy), StorageZone
├── Items/         — CarryCrate, SalesCounter, CounterRepair, RecipeData
├── Machines/      — MachineBase (abstract), Sawmill, Pelletizer, BarkGrinder
├── Managers/      — Save/Audio/Time/Input/Scene/Localization/Economy/Customer/Physics/Market/Upgrade/Sawmill/Worker/Truck/Planting/Fertilize Manager
├── Player/        — PlayerController, PlayerInteraction
├── UI/            — HUD, dialogi, minigry, ustawienia, InventoryPanelUI
├── Vehicle/       — VehicleController, VehicleCamera, VehicleStorage, VehicleHUD
└── Warehouse/     — WarehouseManager, UnloadZone, StorageRack, StorageRackRegistry, StorageManager
```

## Konwencje

### Kod
- **Brak namespace'ów** — flat global namespace
- **4 spacje**, braces Allman style
- Klasy/metody: `PascalCase`, pola: `camelCase`
- Wzorce: ServiceLocator + GameEventSO + ISaveable + Singleton (równolegle)
- SO: `[CreateAssetMenu(menuName = "Timber Tycoon/...")]`

### Assety
- Materiały: `Mat_NazwaOpisu.mat`
- Modele: `NazwaObiektu.fbx`
- Tekstury: `NazwaObiektu_Bake.png` (proceduralne tekstury Blendera NIE eksportują się z FBX — bake do PNG)
- Prefaby: `NazwaObiektu.prefab`
- SO Items: `Item_Typ_Wariant.asset`
- Events: `OnNazwaEventu.asset` w ScriptableObjects/Events/

## Kluczowe decyzje designerskie (nie ruszaj bez rozmowy z Hunterem)
- Single-player, mapa gotowa (brak proc-gen)
- Narzędzia niezniszczalne, tempo jednostajne
- Gracz NIE ma ekwipunku — InventorySystem = rejestr magazynowy
- BEZ: pogody/sezonów, minimapy, random events, difficulty, wildlife
- VFX minimalne: dym, iskry, splash, monety. BEZ trocin/kurzu/liści
- Meble odłożone — typy do ustalenia z Hunterem

---

## Higiena kontekstu

### Kiedy sugerować `/done`
Sugeruj `/done` dokładnie wtedy, gdy Hunter wyrazi satysfakcję z wyniku zadania — rozpoznaj po frazach typu: "super", "dzięki", "działa", "ok", "zatwierdzam", "gotowe", "idealnie", "dobra", "perfect".

NIE sugeruj `/done`:
- Po każdej odpowiedzi automatycznie
- Gdy Hunter wciąż zgłasza poprawki lub iteruje
- Gdy jesteś w środku wieloetapowego zadania

### Kiedy sugerować `/clear` vs `/compact`
- **Przełączenie na inny system projektu** (np. z drzew na UI) → sugeruj `/clear`
- **Kontynuacja tego samego systemu z nowym zadaniem** → sugeruj `/compact Focus on <nazwa systemu>`
- **Po auto-compact** → PIERWSZA akcja: przeczytaj `.claude/checkpoint.md` jeśli istnieje

### Długa sesja z powtórzeniami — kiedy reagować
Gdy w obecnej sesji zauważysz:
- **5+ razy wracałeś do tego samego pliku/systemu z kolejnymi poprawkami**, LUB
- **3+ razy cofałeś własne zmiany** (dodałeś coś, potem Hunter poprosił o usunięcie, potem o powrót, itp.)

Powiedz Hunterowi wprost: "Ta sesja jest już obciążona wieloma iteracjami nad [nazwa systemu]. Kontekst ma dużo szumu ze starych wersji. Zalecam `/checkpoint` + `/compact Focus on [aktualny stan systemu]` przed kolejną zmianą, żeby odciążyć kontekst."

### Ładowanie skills
Ładuj skill TYLKO gdy bieżące zadanie bezpośrednio dotyczy jego tematu:

Domain / asset skills:
- Zadanie o modelu 3D / pipeline Blender→Unity → ładuj `3d-models-assets`
- Zadanie o terenie / środowisku / shaderach mapy → ładuj `map-environment`
- Pytanie o postęp projektu / co dalej → ładuj `progress-tracker`
- Decyzja architektoniczna wymagająca ADR → ładuj `architecture-decision`

Workflow / reference skills (2026-04-24):
- Migracja kodu z jednego systemu do drugiego → ładuj `timber-migration-pattern`
- Usuwanie pliku, klasy, metody, komponentu → ładuj `timber-delete-safety`
- Modyfikacja lub analiza scen (.unity) → ładuj `unity-scene-rules`
- Kod używający StorageManager API → ładuj `storage-manager-api`
- Update checkpoint.md, pre-/clear, pre-/done → ładuj `checkpoint-protocol`
- Współpraca z Hunterem, style komunikacji → ładuj `hunter-communication-style`

Gdy niepewny czy ładować konkretny skill — NIE ładuj i poproś Huntera o potwierdzenie.

---

## Delegowanie do agentów

- **3D modele** (drzewa, budynki, pojazdy, narzędzia, props) → agent `blender-modeler`
- **Teren / mapa / góry / rzeki / drogi** → agent `blender-terrain`
- **Operacje sceny Unity przez Coplay MCP** → agent `unity-operator`

Formuła delegacji: użyj Task tool z promptem zaczynającym od "Use a subagent to...". NIE wykonuj zadań tych agentów w głównej sesji — zawsze deleguj.

---

## Architectural patterns (reusable)

### Diegetic 3D button raycast (ChipperMinigame, 2026-04-28)
Dla minigier z guzikami w 3D scenie (zamiast UI overlay):
- Każdy guzik: osobny GameObject z MeshCollider, nazwy `Button_Green/Red/Yellow`
- Parent machine root: BoxCollider dla E-interaction (Interactable)
- **Podczas minigry: WYŁĄCZ root BoxCollider** — inaczej raycast nigdy nie trafi w guziki
- Po minigrze: WŁĄCZ root collider z powrotem (E-interaction restored)
- Pulse glow: URP/Lit `_EmissionColor` MaterialPropertyBlock (per-instance, nie shared material)
- Raycast layer: domyślny `Physics.DefaultRaycastLayers` pomija `IgnoreRaycast` (layer 2) — upewnij się że guziki nie są na tej warstwie

### Camera lock pattern (minigry, 2026-04-28)
- Zapisz `playerCamera.transform.position` i `.rotation` w world coords (NIE local — local breakuje po cycle reuse bo parent może się ruszyć)
- Lerp do `CameraMinigameTarget` (pusty GameObject, dziecko maszyny, pozycjonowany ręcznie przez Huntera)
- Po minigrze: przywróć world coords bezpośrednio, następnie `SetParent(savedCameraParent)`

### Legacy code conflict po refactorze (lesson, 2026-04-28)
Po dużych zmianach architektonicznych — zawsze sprawdź czy nie zostawiłeś conflicting legacy code:
- Szukaj `Spawn`, `Destroy`, `Force`, `Init` methods w starych klasach które mogą robić to samo co nowy system
- Sprawdź flagi debug (`testMode`, `autoStart`) które mogą aktywować legacy code w Play Mode
- **Case study**: `ChipperMachine.SpawnStump` (legacy physics spawn) + `ChipperMinigameUI.SpawnStumpForPhase1` (kinematic) = dwa pniaki, jeden spada z fizyki, trudne do debugowania

---

## Active sprint

### Sprint Lvl 3: PlankMaker — IN PROGRESS

**Day 1 status (work in progress):**
- ✅ Code skeleton: PlankMakerMachine.cs + PlankMakerMinigameUI.cs (compile clean)
- ✅ Architecture: ProductType enum reused, no new ItemSO
- ⏸️ Asset pipeline: PAUSED — Tripo model needs mechanical redesign
- ❌ Scene setup: deferred (needs model)
- ❌ Smoke test: deferred (needs scene)

**Functional spec (decided):**
- Multi-cut: 3 cuts per log, per-cut PERFECT/GOOD/POOR
- Output per cut: PERFECT 2 planks + 1 bark / GOOD 1+1 / POOR 1+0
- Mouse drag tempo detection (deviation thresholds <5%/5-15%/>15%)
- Tier 1 only this sprint, architecture supports 3 tiers

**Day 2 priorities (next session):**
1. Asset pipeline decision: Blender from scratch vs Tripo retry vs accept current
2. Mechanical clarity issues to address: log cradle (not tracks), visible top rail for sliding head, vertical blade orientation
3. Scene setup with chosen model
4. Smoke test
5. PauseMenuUI ESC integration
6. Output formula verification

---

## Automatyzacja (po akcept Huntera — dotyczy tylko Poziomu 1-2)

Po zaakceptowaniu zadania implementacyjnego przez Huntera:
- Sam wybieraj narzędzia MCP/skills/agentów — nie pytaj którą metodą
- Nowy asset → pełny pipeline: model → tekstury bake → FBX export → Unity import → prefab → kod integrujący
- Nowy system C# → automatycznie dodaj ISaveable + GameEventSO + integrację z ServiceLocator
- Nie przerywaj w środku pełnej implementacji żeby dopytać o szczegóły techniczne (chyba że trafisz na sprzeczne wymagania lub krytyczną niejednoznaczność, która może popsuć system)
```

## Section 6: Custom commands

### C:\Users\<user>\.claude\commands\
FOLDER_MISSING

### D:\Unity\Timber_Tycoon\.claude\commands\checkpoint.md

```md
---
name: checkpoint
description: Smart context preservation before compact. Use between task phases or when session is getting long.
user-invocable: true
allowed-tools: Read, Write
---

Context checkpoint — preserve important state before compacting.

## Step 1: Identify What to Preserve

Review the current conversation and identify:
- Files that were modified in this session (list with full paths)
- Key decisions that were made (architectural choices, naming conventions, values chosen)
- Any bugs found or fixed (with root cause)
- Current state of the system being worked on (what works, what doesn't, what's half-done)

## Step 2: Write Checkpoint File

Write the checkpoint to `.claude/checkpoint.md`, overwriting any previous checkpoint:

```markdown
# Checkpoint — <YYYY-MM-DD> — <system/topic name>

## Context sesji
<1-2 sentences describing what was worked on>

## Modified Files
- <full path 1>
- <full path 2>
(etc.)

## Key Decisions
- <decision 1 with brief rationale>
- <decision 2 with brief rationale>

## Current State
- Working: <what works>
- Broken: <what doesn't work>
- Half-done: <what's in progress>

## Next Steps
1. <immediate next action>
2. <subsequent actions>

## Zachowane ograniczenia
<any constraints or "don't do X" rules that must survive compact>
```

Use today's actual date.

## Step 3: Tell Hunter What to Run

After writing the checkpoint, instruct Hunter:

"Checkpoint zapisany. Uruchom teraz:
`/compact Focus on: <list the 2-3 most important things from the Current State section>`

Po kompakcji pierwsza akcja Claude'a będzie przeczytać ten checkpoint, więc nic istotnego się nie zgubi."
```

### D:\Unity\Timber_Tycoon\.claude\commands\done.md

```md
---
name: done
description: Run after completing a task when Hunter confirms satisfaction. Updates progress, backs up scene, commits, and prepares for next task.
user-invocable: true
allowed-tools: Read, Write, Glob, Grep, Bash
---

Task completion workflow. Execute ALL 6 steps IN THIS ORDER. Do not skip.

## Step 1: Summarize What Was Done

Write a brief summary (3-5 bullets) of what was accomplished in this task. Focus on concrete changes, not effort spent.

## Step 2: Update Progress Tracker

Read `.claude/skills/progress-tracker/SKILL.md` and update the relevant checkboxes/status if a tracked item was completed.

Additionally, if during this task:
- A new map asset was created/modified → update `.claude/skills/map-environment/SKILL.md`
- A new 3D model was created/modified → update `.claude/skills/3d-models-assets/SKILL.md`

## Step 3: Scene Backup

Create a timestamped backup of the Unity scene:

```bash
mkdir -p Assets/_Backup_<YYYY-MM-DD>/
cp Assets/Demo_Scene.unity "Assets/_Backup_<YYYY-MM-DD>/Demo_Scene_<HHMM>.unity"
```

Use today's actual date (YYYY-MM-DD format) and current time (HHMM format, 24-hour). NEVER overwrite an existing backup — if the filename already exists, increment minute by 1 until unique.

## Step 4: Git Commit (only if files were modified)

If code or asset files were modified during this task:

```bash
git add -A
git commit -m "<type>: <concise description>"
```

Where `<type>` is one of: `feat`, `fix`, `chore`, `refactor`, `docs`, `style`. Use conventional commits format.

If NO files were modified (pure discussion/planning session) → skip this step entirely, do not create an empty commit.

## Step 5: Context Hygiene Recommendation

Choose ONE recommendation based on what Hunter said about the next task:

**If next task is RELATED to current system** (same files, same area):
→ Tell Hunter: "Sugeruję `/compact Focus on <nazwa systemu> changes` żeby odchudzić kontekst, ale zachować istotne info."

**If next task is UNRELATED** (different system, different area):
→ Tell Hunter: "Sugeruję `/clear` — następne zadanie dotyczy innego systemu i świeży kontekst da lepsze wyniki."

**If current session has 5+ iterations on the same file/system** (regardless of next task):
→ Tell Hunter: "Ta sesja jest już długa i obciążona wieloma iteracjami. Zdecydowanie `/clear` przed kolejnym zadaniem."

**If Hunter hasn't said what's next yet** → skip to Step 6 without making a recommendation.

## Step 6: Ask What's Next

Ask Hunter what chce robić dalej. Include 2-3 suggestions from progress-tracker showing items marked `⬜` (not started) or `🔨` (in progress) that are logical next steps given what was just completed.
```

## Section 7: Existing SKILL.md files

No SKILL.md files found under C:\Users\<user>\.claude\skills\

### D:\Unity\Timber_Tycoon\.claude\skills\3d-models-assets\SKILL.md

```md
---
name: 3d-models-assets
description: Szczegóły modeli 3D — pojazd, drzewa (4 gatunki), narzędzia, tartak, lada, firewood, carry crate, storage rack. Używaj gdy pracujesz nad modelami 3D, materiałami, prefabami, teksturami lub pipeline'em Blender→Unity.
---

# Modele 3D — Szczegóły

## Pojazd (Kei Truck) ✅
- Blend: `Assets/Models/Cars/Car1_KeiTruck.blend`
- FBX: `Assets/Models/Cars/Car1.fbx` (620 verts, 42 dzieci)
- **Forward = -transform.right** (quirk eksportu FBX)
- VehicleController: `forwardAxis = Right`, jawny `rb.centerOfMass`
- Materiały: 10 double-sided (`_Cull = 0`)
- RebuildCarModel.cs do przebudowy z FBX
- maxSpeed: 22.5, acceleration: 12
- Reflektory pod HeadL/HeadR

## NPCPickup01 ✅ (2026-04-16)
- Blend: `Assets/Models/Cars/NPC_Car_1/NPCPickup01.blend`, FBX: `Assets/Models/Cars/NPCPickup01.fbx` (210 KB, 5 meshów: main body + 4 wheels)
- Wymiary: 4m × 1.85m × 1.52m, working-truck / farm pickup style
- **Hierarchia**: NPCPickup01 (root) → WheelLF, WheelRF, WheelLR, WheelRR jako child objects, origin każdego koła w środku piasty (axle). VehicleController.FindWheels rozpoznaje po prefiksie `Wheel` + sufiks F/R (F = przednie, skręcają)
- Axis konwencja: +X = front, +Y = **LEWO**, -Y = **PRAWO** (z perspektywy kierowcy)
- **9 materiałów** (URP/Lit) w `Assets/Models/Cars/Materials/`: Mat_Body (variable, MPB randomized), Mat_Trim, Mat_Glass, Mat_Mirror, Mat_Tire, Mat_Rim, Mat_Chassis, Mat_Headlight (emission 0.6), Mat_Taillight (emission 0.4)
- **4 baked tekstury PNG** w `Assets/Models/Cars/Textures/`: Mat_Tire_Diffuse (512, rubber grain), Mat_Rim_Diffuse (512, brushed metal, Metallic=0.15), Mat_Body_Diffuse (1024, subtle dirt overlay, biały-dominant dla MPB), Mat_Chassis_Diffuse (512, metal wear + rdzawe plamki)
- **CarVariants SO** (`Assets/Project/ScriptableObjects/CarVariants.asset`): 1 prefab (NPCPickup01.fbx) + 8 kolorów karoserii (#A8433A, #3F5E78, #D9CDB0, #3C5641, #B5683A, #C69A3A, #6F6A62, #4A3B2E)
- **Bootstrap**: `NPCVehicleTestBootstrap` spawnuje z CarVariants, MPB `_BaseColor` nadpisuje Mat_Body przy spawnie. `bodyMaterialName = "Mat_Body"`
- Editor scripts: `SetupNPCPickup01Materials.cs` (tworzy/konfiguruje materiały + remap FBX), `SetupNPCCarVariants.cs` (wypełnia SO pulą i hookuje bootstrap)

## Drzewa — 4 gatunki ✅
Każdy gatunek ma pełny cykl: modele (Adult/Stump/Trunk/Log/Sapling × 4 rozmiary), TreeTypeData SO, prefaby, Item_Log/Plank, Recipe_Sawmill, Firewood FBX+prefab+materiały, drzewa na scenie.

| Gatunek | Bark | Korona | Uwagi |
|---------|------|--------|-------|
| Spruce (Świerk) | prosty | vertex color gradient + noise | baseline |
| Birch (Brzoza) | Voronoi baked PNG | vertex color gradient + noise | birch spots Scale 6 |
| Oak (Dąb) | baked | Z×1.35 + X×0.80 | 5 drzew na scenie |
| Maple (Klon) | baked | Z×1.20 | 5 drzew na scenie |

- **Log scale inconsistency**: Spruce_Log Scale 100, Birch_Log Scale 1 → VehicleStorage kompensuje via `data.prefab.transform.localScale`
- **Planowane**: Acacia, Mahogany (po demo)

## FlatbedCargo v2 ✅ (2026-04-12)
- FBX: `Assets/Models/Cars/FlatbedCargo.fbx` (506v), Blend: `FlatbedCargo.blend`
- 3 cumulative fill meshes: Fill_1 (1-33%), Fill_2 (34-67%), Fill_3 (68-100%)
- Fill_1: 4 kłody + 3 pniaki, Fill_2: +3 kłody + 2 pniaki, Fill_3: +2 kłody + 2 pniaki + 2 liny
- 7 materiałów: 3× bark (Spruce/Birch/Oak) + 3× cross-section + 1× vertex color details
- Mapowanie po nazwie Blender→Unity w SetupFlatbedCargo.cs (nie po indeksie!)
- Pniaki pionowe w kompaktowej grupce przy Z+ końcu, drugie piętro na fill 2/3
- Liny na Fill_3: X=±0.32/0.24, Y śledzi kontur stosu
- Unity transform: localPos user-set, rot=(180,90,0), scale=1.5
- Skrypty: `create_flatbed_cargo_v2.py` (build), `export_flatbed_cargo.py` (join+export)
- Edytowalność: skrypt generuje osobne obiekty per pniak/lina, user przesuwa w Blenderze
- MeshCollider na każdym fill level

## Narzędzia ✅
- Siekiera: `Assets/Models/Tools/Axe_T1.fbx` (20v, broda+policzki+obuch)
- Łopata: `Assets/Models/Tools/Shovel.fbx`

## Tartak (Sawmill) ✅
- `Assets/Models/Sawmill.fbx` (416v, 292f), 23×17m
- Pozycja: (177.9, 7.62, -88.71) — NIE ZMIENIAĆ
- Mat_Sawmill (VertexColorLit, Brightness=0.35)
- Layout: maszyny NW | magazyn NE | meble SW | lada SE

## Lada (Counters) ✅
- `Assets/Models/Counters.fbx` — Counter_Fixed (152v) + Counter_Broken (80v)
- CounterRepair.cs przełącza widoczność (hold E)

## Firewood ✅ (2026-04-11 unifikacja)
- Species-agnostic: `StorageFamily.Firewood`, 1 BasicFirewood zamiast 4 per-species
- 3 modele: `Firewood_Basic/Fine/Premium.fbx` (40v/44f)
- Procedural Wave RINGS z ciągłością słojów endcap↔flat cut
- Baked diffuse + normal PNG
- Prefab `Firewood_Basic.prefab` z CollectableFirewood

## FirewoodRack v2 ✅ (2026-04-11)
- U-shape model 168v/126f, wymiary 0.9×0.45×0.65m
- 3 fill warstwy (5 klinków × 3 = 15 widocznych przy 100%)
- Multi-layer wood texture baked 1024×1024
- 3 vertex color tints (słupki ciemne / belki ścian / półki jasne)
- Mat_FirewoodRack_Frame URP/Lit

## StumpRack v2 ✅ (2026-04-12)
- Pallet bin z koziołkami, baked frame texture + per-species bark/cross-section
- FBX: `Assets/Models/Environment/StumpRack.fbx` (636v, 19 meshów + 3 empties)
- Wymiary: 1.6m × 1.2m × 0.9m
- 10 pniaków (4+3+3): Fill_1 w rogach, Fill_2 w środku, Fill_3 na górze
- 3 vertex rings per pniak (root flare + collar + top cut) — naturalna sylwetka
- Gatunki: Oak=2, Birch=3, Spruce=3, Maple=2 (size_scale 0.80-1.00)
- **1 materiał per mesh** (wzorzec LogRack v2)
- 9 materiałów: 4× bark + 4× cross-section + 1× baked frame (`StumpRack_Frame_Diffuse.png` 512×512)
- Skrypt: `create_stump_rack_v2.py`, setup: `SetupStorageRacks.SetupStumpRack` (NIE nadpisuje materiałów)

## Shredder (Szreder) ✅ (2026-04-20)
- Bazowy mesh: Tripo AI (monolityczny), podzielony w Blenderze na 3 meshe + 2 empty
- Blend: `Assets/Project/Models/Shredder/Shredder_Split.blend`
- FBX: `Assets/Project/Models/Shredder/Shredder.fbx`
- Hierarchia: Shredder_Root → Body_Main (408v) + Shaft_Left (1074v) + Shaft_Right (1040v) + BagSpawnPoint (empty) + InteractionPoint (empty)
- Vertex colors: frame #6B6B70, shafts #4A4A50, button_red #CC3300, button_green #336600, button_yellow #CC9900, orange accent #CC5500 (23 face'y zaznaczone ręcznie przez Huntera)
- Podział: flood-fill BFS po krawędziach → 3 izolowane wyspy. Frame = NAJMNIEJSZA wyspa (88v).
- Unity materiał: `Mat_Shredder_VertexColor` (Custom/VertexColorLit, _Brightness=1.0)
- Pozycja w scenie: (175, 8, -108), rot Y=180
- Animacja: `TEST_ShaftRotation.cs` — Vector3.forward (Blender Y → Unity Z), Space.Self, 180°/s, wały ku środkowi
- Setup: `Assets/Editor/SetupShredderAll.cs` (menu: Timber Tycoon/Setup Shredder All)

## BagRack ✅ (2026-04-20)
- Stojak na torby/chips, 3 półki × 3 sloty = 9 miejsc
- Blend: `Assets/Project/Models/BagRack/BagRack_Blender.blend`
- FBX: `Assets/Project/Models/BagRack/BagRack.fbx` (192 poly, 8 meshów + 10 empties)
- Wymiary: 2.4m × 0.7m × 2.2m
- Hierarchia: BagRack_Root → Post_1-4, Shelf_1-3, BackPanel + SlotAnchor_X_Y × 9 + InteractionPoint
- Tekstura: `BagRack_Frame_Diffuse.png` 512×512, Cycles bake proceduralny (Wave+Noise)
  - Posty: gunmetal gray z scratch noise
  - Półki: pine/sosna słoje drewna
  - BackPanel: walnut ciemne drewno
- Materiał Unity: `Mat_BagRack_Frame` URP/Lit
- Pozycja w scenie: `Tartak_Area/BagRack` (175.5, 8.0, -103.754) rot Y=90
- Setup: `Assets/Editor/SetupBagRack.cs`

## PelletBag ✅ (2026-05-15)
- Worek peletu 15kg, low-poly bag shape z gusset fin na górze
- Build script: `Assets/Art/BlenderScripts/build_pelletbag.py` (v8 — manual atlas, no Cycles bake)
- FBX: `Assets/Models/Storage/PelletBag.fbx` (206v, 202f, 24 KB)
- Wymiary: 0.30W × 0.18D × 0.41H m, pivot na DOLE (origin Z=0 w Blenderze)
- **Atlas podejście**: numpy composite — label PNG (Tripo) w lewych 75% (0..0.75 U), body solid off-white w prawym-dolnym (0.875, 0.25), accent tan w prawym-górnym (0.875, 0.75)
- Label PNG: `Assets/Textures/Pellets/T_PelletBag_Label.png` (Tripo AI — drzewko + pellets + "15")
- Atlas PNG: `Assets/Textures/Pellets/T_PelletBag_Atlas.png` (1024×1024 sRGB, BC7 w Unity)
- Materiał Unity: `Assets/Models/Storage/Materials/M_PelletBag_Baked.mat` (URP/Lit, single slot)
- Prefab: `Assets/Project/Prefabs/BagOfPellet.prefab` (1 mat slot)
- Accent strefa: Z=0.33-0.36 w Blenderze (wąski band przy górze worka)
- **UWAGA**: bag X columns w PelletRack: ±0.700 (vs BagRack SlotAnchor ±0.747 — diff 4.7cm)

## PelletRack_Placeholder ✅ (2026-05-15)
- Scena: `Tartak_Area/PelletRack_Placeholder` (177.0, 8.0, -98.0), rot Y=0, scale (1,1,1)
- Komponenty: StorageRack (family=PelletBag, maxCapacity=9), StorageRackInteractable
- **RackVisual**: child z BagRack.fbx (BackPanel + Post_1-4 + Shelf_1-3 + Mat_BagRack_Frame)
  - FBX-owned Fill_N i SlotAnchor_N_M stripped przy setupie
  - Shelf Y locals (identyczne z BagRack): 0.15 / 0.90 / 1.65
  - SlotAnchor surface Y: 0.175 / 0.925 / 1.675
- **Fill groups** (row-based — każdy = 1 półka):
  - Fill_1 = dolna półka (Y=0.175): 3× BagOfPellet na X=−0.7/0/+0.7
  - Fill_2 = środkowa półka (Y=0.925): 3× BagOfPellet na X=−0.7/0/+0.7
  - Fill_3 = górna półka (Y=1.675): 3× BagOfPellet na X=−0.7/0/+0.7
- Fill groups domyślnie INACTIVE — StorageRack aktywuje w runtime

## LogRack v2 ✅ (2026-04-12)
- Log Bunk z koziołkami (A-frame supports), 3 pary pochylonych słupków
- FBX: `Assets/Models/Environment/LogRack.fbx` (700v, 21 meshów + 3 empties)
- Wymiary: 2.2m × 1.07m × 0.90m (długość × szerokość × wysokość)
- 3 fill levels (Fill_1: 7, Fill_2: 6, Fill_3: 5 kłód) — hexagonalne układanie
- **1 materiał per mesh** — eliminuje reordering slotów w Unity
- Struktura: Fill_X → Fill_X_BarkSpecies + Fill_X_CrossSpecies (osobne meshe per gatunek × typ)
- 9 materiałów: 4× bark (Spruce/Birch/Oak/Maple) + 4× cross-section + 1× baked frame
- Baked frame texture: `LogRack_Frame_Diffuse.png` (512×512, Cycles procedural noise)
- Skrypt budowy: `create_log_rack_v2.py`, setup: `SetupLogRackV2.cs`
- **UWAGA**: `SetupStorageRacks.cs` i `SetupStorageRackMaterial.cs` NIE nadpisują materiałów LogRacka

## CarryCrate v2 ✅ (2026-04-11)
- 3 FBX: Small/Medium/Large (10/15/25 slotów)
- Single welded mesh 32v/26f, handles na krótkich ścianach
- Vertex colors per region + procedural noise
- CrateContentRenderer spawnuje firewood per slot z diff-refresh

## Ikony UI ✅
- 30/30 ikon jako low-poly 3D przez IconRenderer.cs
- Pipeline: `IconRenderer.Execute()` → `AssignGeneratedIcons.AssignAll()`
- Logi (6), Deski (6), Narzędzia (5), Bark (1), Sawdust×2, Pellet×3, Mulch (1), Meble×6

## PlankRack ✅ (2026-05-03)
- Blend: `_BlenderOutputs/PlankRack/PlankRack.blend`
- FBX: `Assets/Project/Models/Racks/PlankRack/PlankRack.fbx` (40060 bytes)
- Wymiary: Frame 0.9×0.64×1.22m, 3 półki (Fill_1/2/3) z 4 slotami każda
- Hierarchia: PlankRack → {Frame, Arms, Fill_1, Fill_2, Fill_3} → Fill_N_Slot_M (PLAIN_AXES empties)
- Planki: `Plank_Spruce.prefab` (`Assets/Models/Planks/Plank_Spruce.fbx`) instancjonowane w runtime przy każdym slocie (nie w FBX)
- **UWAGA bake_space_transform bug**: empties dostają Lcl Rotation=90°X + raw Blender positions. Fix w `SetupPlankRackFull.cs`: remap slot.localPos `(X,Y,Z)→(X,Z,-Y)` + reset rotation do identity przed InstantiatePrefab
- Materiały: `Mat_LogRack_Frame` (frame texture), `Mat_PlankRack_Arm` (arm texture), `Mat_Plank_Spruce`
- Setup: `Assets/Editor/SetupPlankRackFull.cs` (menu: Timber Tycoon/Setup PlankRack Full Integration)
- StorageRack: family=Plank, maxCapacity=60, fillLevel1/2/3 wired do Fill_1/2/3

## Pelletizer 🔨 (2026-05-05 — textury baked, FBX pending)
- Blend: `_BlenderOutputs/Pelletizer/Pelletizer_Source_WIP.blend`
- FBX: nie wyeksportowany jeszcze
- Hierarchia (30 mesh'y + 1 empty root): Pelletizer_Root → Body, Hopper, Collar, Base, Motor + children (Fins, FanCover, Shaft, TerminalBox, Coupling), DiePlate, Rollers×3, Arms×3, Spindle, Flanges×4, Chute, Panel, Buttons×3, Motor_Mount
- **Motor_Mount** (2026-05-05): cuboid 0.20×0.30×0.0811m, zastąpił Foot_L/R. Siedzi dokładnie między Base top (Z=0.05) a cylinder ring (Z=0.1311)
- **Materiały — 10 aktywnych** (wszystkie procedural → baked 1024×1024 PNG):
  - `BodyPaint_Red`: slate blue cynkowany (#1E2733), 11 mesh'y. R=0.70, M=0.30
  - `Galvanized`: jasny szary (#9CA0A3 z rdzą), 4 mesh'e. R=0.55, M=0.65
  - `Bed_Rollers`: ciemny żelazny, 4 mesh'e. R=0.50, M=0.70
  - `Blade`: prawie czarny polished, 6 mesh'y. R=0.30, M=0.85
  - `Panel_Body`: steel, 2 mesh'e. R=0.70, M=0.30
  - `HeavyRust`: rdzawy (0 użyć, zasób). R=0.80, M=0.25
  - `Button_Green/Red/Yellow`: solid emission, 1 mesh każdy (NIE baked)
  - Zachowane bez użyć: `Frame_Steel`, `Head_Body`, `Head_Mount`, `Bed_Stops`
- **PNG tekstury** (1024×1024 sRGB): `Assets/Project/Textures/Tier1/Pelletizer/*.png` (6 plików ~420-1021 KB)
- **Backup pre-bake**: `_BlenderOutputs/Pelletizer/backups/Pelletizer_Source_WIP_2026-05-05_pre_bake.blend`
- **Procedural backups**: 6 materiałów jako `*_PROCEDURAL_BACKUP` w bpy.data.materials
- **UWAGA cylinder Z bug**: motor cylinder bottom ring = Z=0.1311 (10 verts), NIE Z=0.1250 (5 verts = outlier). Zawsze używaj histogram Z z count>=8 dla prawdziwego bottom ring.

## Pipeline Blender → Unity
1. Model w Blenderze (real-world scale, 1 unit = 1m)
2. UV Unwrap: Smart UV Project
3. Proceduralne tekstury → **MUSZĄ być baked do PNG** (FBX nie eksportuje proceduralnych)
4. Bake: Cycles, Diffuse Color Only, 512×512 lub 1024×1024
5. FBX export: `FBX_SCALE_ALL`, `bake_space_transform=True`, `mesh_smooth_type='OFF'`
6. Unity import: scale (1,1,1), materiały assign baked PNG
7. Prefab + konfiguracja komponentów
```

### D:\Unity\Timber_Tycoon\.claude\skills\architecture-decision\SKILL.md

```md
---
name: architecture-decision
description: "Creates an Architecture Decision Record (ADR) documenting a significant technical decision, its context, alternatives considered, and consequences. Every major technical choice should have an ADR."
argument-hint: "[title]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write
---

When this skill is invoked:

1. **Determine the next ADR number** by scanning `docs/architecture/` for
   existing ADRs.

2. **Gather context** by reading related code and existing ADRs.

3. **Guide the user through the decision** by asking clarifying questions if
   the title alone is not sufficient.

4. **Generate the ADR** following this format:

```markdown
# ADR-[NNNN]: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXXX]

## Date
[Date of decision]

## Context

### Problem Statement
[What problem are we solving? Why does this decision need to be made now?]

### Constraints
- [Technical constraints]
- [Timeline constraints]
- [Resource constraints]
- [Compatibility requirements]

### Requirements
- [Must support X]
- [Must perform within Y budget]
- [Must integrate with Z]

## Decision

[The specific technical decision made, described in enough detail for someone
to implement it.]

### Architecture Diagram
[ASCII diagram or description of the system architecture this creates]

### Key Interfaces
[API contracts or interface definitions this decision creates]

## Alternatives Considered

### Alternative 1: [Name]
- **Description**: [How this would work]
- **Pros**: [Advantages]
- **Cons**: [Disadvantages]
- **Rejection Reason**: [Why this was not chosen]

### Alternative 2: [Name]
- **Description**: [How this would work]
- **Pros**: [Advantages]
- **Cons**: [Disadvantages]
- **Rejection Reason**: [Why this was not chosen]

## Consequences

### Positive
- [Good outcomes of this decision]

### Negative
- [Trade-offs and costs accepted]

### Risks
- [Things that could go wrong]
- [Mitigation for each risk]

## Performance Implications
- **CPU**: [Expected impact]
- **Memory**: [Expected impact]
- **Load Time**: [Expected impact]
- **Network**: [Expected impact, if applicable]

## Migration Plan
[If this changes existing code, how do we get from here to there?]

## Validation Criteria
[How will we know this decision was correct? What metrics or tests?]

## Related Decisions
- [Links to related ADRs]
- [Links to related design documents]
```

5. **Save the ADR** to `docs/architecture/adr-[NNNN]-[slug].md`.
```

### D:\Unity\Timber_Tycoon\.claude\skills\checkpoint-protocol\SKILL.md

```md
---
name: checkpoint-protocol
description: Checkpoint.md maintenance — kiedy update, co zawsze ma być, pre-/clear checklist, anti-patterns. Używaj gdy aktualizujesz checkpoint lub przed /clear, /compact, /done.
---

# Checkpoint Protocol — Timber Tycoon

`checkpoint.md` = kontrakt między sesjami. Pierwsza rzecz którą nowa sesja czyta. Jeśli nie jest aktualny → nowa sesja startuje z błędnym obrazem projektu.

## Kiedy update'ować

- **Po zakończeniu sprint'u** — commit sprint'a + update checkpoint w tym samym batch'u
- **Przed `/compact`** — upewnij się że current state jest zapisany
- **Przed `/clear`** — KRYTYCZNE: po /clear context znika bezpowrotnie
- **Przed `/done`** — last chance zapisać closing state sesji
- **Po istotnej decyzji architekturalnej** — żeby następna sesja ją znała
- **Po housekeeping** — .gitignore change, folder delete, plugin install/removal

**NIE update:**
- Po każdym małym commit'cie (overkill)
- Gdy zmieniasz jeden log message lub trywialny parametr

## Struktura checkpoint.md — wymagane sekcje

W tej kolejności:

1. **Header** — status + data + session summary (1-2 linie)
2. **Session results** — co zrobione w tej sesji (lista sprintów/tasków)
3. **Commits** — hashes + opisy (chronologicznie, najnowsze na górze)
4. **Current project state** — aktualny stan systemów (single source of truth, liczba skryptów, etc.)
5. **Known minor issues** — drobne problemy, benign warnings
6. **Pending** — co do zrobienia następnym razem (priorytetyzowane)
7. **Architectural decisions** — decyzje "don't revisit" (np. StorageManager jedyny authority, NPC vehicle-only)
8. **Files to read first next session** — priority order

## Co ZAWSZE ma być

- **Commit hashes** dla każdej zamkniętej pracy: `abc1234` — opis
- **Stan systemu** z liczbami: `StorageManager = single source, scripts: 163 → 151`
- **Kluczowe API** używane często (StorageManager.GetAmount, AddToStorage, etc.)
- **Cross-references** do innych docs (REFACTOR_PROGRESS, SYSTEMS_MAP, SCRIPTS_INVENTORY)
- **Collaboration model** reminder: Hunter = Creative Director (nie programista), implementacja = Claude

## Co NIE ma być

- Historii rozmów ("wczoraj mieliśmy problem z...")
- Rozważań "co zrobić" — te należą do sekcji Pending
- Draft'ów niezatwierdzonych zmian
- Duplikacji treści z docs/ plików — checkpoint to summary, nie pełna dokumentacja
- **Outdated instructions** — jeśli akcja wykonana, USUŃ ją lub oznacz RESOLVED

## Pre-/clear checklist

Przed `/clear` zweryfikuj WSZYSTKIE punkty:

- [ ] Sprint commits udokumentowane (hashes obecne)
- [ ] Outdated sections usunięte (np. "ACTION REQUIRED" jeśli już zrobione)
- [ ] Architectural decisions zapisane jeśli ustalone w sesji
- [ ] Pending list zaktualizowana (dodaj co zostało, usuń co zrobione)
- [ ] Cross-references działają (pliki które wspominasz istnieją w repo)
- [ ] Last `git commit` = po ostatniej zmianie checkpoint

**Jeśli checklist niekompletna → NIE /clear.** Utrata nieodwracalna.

## Anti-patterns z praktyki

**Pozostawienie outdated action items:**
```
❌ "Missing script — ACTION REQUIRED by Hunter"  (po tym jak Hunter już to zrobił)
✅ "Missing script: RESOLVED (Hunter removed component 2026-04-24)"
```

**Generic pending bez priorytetu:**
```
❌ "Sprint A — do zrobienia kiedyś"
✅ "Sprint A (~2-3 sesje) — Upgrade Shop + Worker Scheduler + Day-End Tick. Wymaga UX decisions od Huntera."
```

**Historyczne szczegóły zamiast decyzji:**
```
❌ "Długo dyskutowaliśmy o ścieżce A vs B, ostatecznie wybraliśmy B bo..."
✅ "Decyzja: Ścieżka B — delete unity-dev-toolkit (Dev-GOM, experimental, no license)"
```

**Brak commit hashes:**
```
❌ "(pending) — housekeeping commit"
✅ "`1ca687a` — chore: housekeeping (.gitignore + checkpoint + progress-tracker)"
```

## Update workflow

1. Read `checkpoint.md` (pełne — nie skip)
2. Identify sections do zmiany
3. Edits:
   - **Prepend** do "Session results" jeśli nowa sesja
   - **Update in-place** sekcje z przestarzałymi info
   - **Append** do Pending jeśli nowe items
   - **Remove** outdated action items / RESOLVED issues
4. Verify cross-references — pliki wspomniane istnieją na dysku
5. Commit:

```bash
git add checkpoint.md
git commit -m "docs: update checkpoint — [summary of what changed]"
```

## Relationship to other docs

| Plik | Rola | Kiedy update |
|------|------|-------------|
| `checkpoint.md` | SUMMARY state, reading-first, short | po każdym sprincie / przed /clear |
| `Docs/REFACTOR_PROGRESS.md` | DETAILED history, append-only log | po każdym sprincie refaktoru |
| `Docs/SCRIPTS_INVENTORY.md` | SNAPSHOT scripts count/locations | gdy liczba skryptów się zmienia |
| `Docs/SYSTEMS_MAP.md` | ARCHITECTURE reference | gdy nowy system dodany |

Checkpoint **references** te pliki — NIE duplikuje ich treści.

## Powiązane skills

- `hunter-communication-style` — collaboration model section jest core checkpoint'u
- `timber-migration-pattern` — każdy sprint ma commit hash do udokumentowania
- `timber-delete-safety` — każdy delete sprint = osobna sekcja w REFACTOR_PROGRESS
```

### D:\Unity\Timber_Tycoon\.claude\skills\map-environment\SKILL.md

```md
---
name: map-environment
description: Szczegóły mapy Timber Tycoon — teren, góry, rzeka, klif, wodospad, jaskinia, most, drogi, shadery, setup scripty. Używaj gdy pracujesz nad mapą, środowiskiem, terenem, modelami environment lub ich shaderami.
---

# Mapa — Szczegóły Środowiska

## Architektura mapy
- Teren, góry, klif, jaskinia, most — osobne FBX/obiekty (NIE wbudowane w teren)
- Eksport: `FBX_SCALE_ALL` + `bake_space_transform=True` → Unity scale (1,1,1)
- Agenty Blender: `.claude/agents/blender-terrain.md`, `.claude/agents/blender-modeler.md`
- Plik Blendera: `Assets/Models/Environment/Terrain.blend`

## Pozycjonowanie mapy
- LowPolyTerrain na (0,0,0) scale (1,1,1) — bakowane współrzędne z Terrain.blend
- Wszystkie obiekty mapy na bakowanych pozycjach z FBX
- Tartak (Sawmill): pozycja (177.9, 7.62, -88.71), rotation Y=0 — NIE ZMIENIAĆ
- Setup scripty: `RebuildFullMap.cs` (master) → uruchamia wszystkie poniższe
- **Prędkość gracza (tymczasowo ×10):** walkSpeed: 5→50, runSpeed: 8→80

## Skrypty setup mapy
1. RebuildFullMap.cs — master
2. PlaceMountains.cs — góry przednie
3. SetupBackdropAndCover.cs — góry tylne + zakrycie rzeki
4. SetupRiver.cs — rzeka + RiverZone
5. SetupCliff.cs — klif z wodospadem
6. SetupRiverCave.cs — jaskinia rzeki
7. SetupWaterfall.cs — wodospad
8. SetupBridge.cs — most
9. SetupSawmill.cs — tartak
10. SetupRoads.cs — drogi z teksturami

## Obiekty mapy (wszystkie ✅)

**Teren**: LowPolyTerrain.fbx (98k verts, 2m spacing), 600×650m, vertex colors (sRGB)
**Góry przednie**: Mountains.fbx (8.2k verts), pierścień dookoła mapy, 40-90m
**Góry tylne**: Mountains_Backdrop.fbx (2100 verts), 100-151m, Mat_Backdrop (VertexColorLit)
**Góry wewnętrzne v2**: 30 gór, 7 profili, 5 warstw shader (MountainLayered)
**Rzeka**: River.fbx (1017v), semi-elipsa, shader Custom/LowPolyWater, WaterFlow.cs + WaterZone.cs
**Klif**: Cliff_Waterfall.fbx (640v), jaskinia u podnóża, MeshCollider, pozycja: NIE ZMIENIAĆ
**Wodospad**: Waterfall.fbx (140v), shader Custom/Waterfall, pozycja: NIE ZMIENIAĆ
**Most**: Bridge.fbx (996v), łuk paraboliczny 56m, Unity X=120 Z=152
**Jaskinia rzeki**: River_Cave.fbx (556v), tunel z otworem 18m×8m
**Zakrycie rzeki**: River_CaveCover.fbx (56v)
**Drogi**: Roads.fbx (4 obiekty: Road_Sawmill_Bridge v2 dwupasmowa 6m + Road_Logging_Dirt + Road_Center_Dirt + Road_CenterLine), shader Custom/RoadTextured + Mat_Road_CenterLine. Wszystkie drogi face-copy topology z terrainu +2cm offset. Teren lokalnie obniżony pod drogami. Roads.blend zapisany (2026-04-13)
**Tartak**: Sawmill.fbx (416v), 23×17m, Mat_Sawmill (VertexColorLit, Brightness=0.35)

## Drogi — W TOKU (3/6-8)
- Gotowe: Road_Sawmill_Bridge v2 (żwir 6m dwupasmowa + linia środkowa), Road_Logging_Dirt (polna 3.5m), Road_Center_Dirt (polna 3.5m) — wszystkie z conformity do terrainu (face-copy +2cm)
- Brakuje: droga asfaltowa (miasto), ścieżka leśna (nad rzeką)

## Shadery custom
- `Custom/LowPolyWater` — animowane fale, flow, fresnel, piana (rzeka)
- `Custom/Waterfall` — vertex colors + animacja (fale vertex + scrolling foam), Cull Off
- `Custom/VertexColorLit` — vertex colors + oświetlenie, Cull Off, BEZ animacji (budynki)
- `Custom/RoadTextured` — world-space XZ projekcja + normal map + vertex color blend
- `Custom/RockTriplanar` — triplanar projection dla skał
- `Custom/MountainLayered` — 5-warstwowy shader gór wewnętrznych
- `Custom/TerrainTextured` — tekstury terenu

## Oświetlenie
- Day/Night Cycle v2
- 10 shaderów Forward+ compatible (LIGHT_LOOP_BEGIN)
- 24 lampy (16 street + 8 sawmill) z ArtificialLight + LightingManager
- Reflektory auta pod HeadL/HeadR

## Rampa rozładunkowa
- Pozycja: (163.04, 8.0, -96.71), rot (270, 0.67, 0), scale 0.5
```

### D:\Unity\Timber_Tycoon\.claude\skills\progress-tracker\SKILL.md

```md
---
name: progress-tracker
description: Status postępu Timber Tycoon — fazy 1-4, co zrobione, co w toku, co następne. Używaj gdy sprawdzasz postęp, planujesz kolejne zadania, lub aktualizujesz status po zakończeniu pracy.
---

# Timber Tycoon — Postęp

## Faza 1 — KOMPLETNA ✅
10 systemów: ServiceLocator, GameEventSO, GameStateMachine, InputManager, SaveManager, AudioManager, SceneTransitionManager, LocalizationManager, TimeManager, PhysicsOptimizer

## Faza 2 — KOMPLETNA ✅
14 systemów upgraded z ISaveable + GameEventSO (ChoppableTree, GrowingTree, VehicleController, VehicleStorage, MachineBase, InventorySystem, ~~WarehouseManager~~ [USUNIĘTY 2026-04-24 → zastąpiony StorageManager], QuestManager, CounterRepair, PlayerController, PlayerInteraction, AudioManager, LocalizationManager, ToolManager)

## Faza 3 — PRAWIE KOMPLETNA (~95%)
- ✅ Carry System, Narzędzia, Processing Chain (MachineController + MachineOutputCalculator)
- ✅ Chopping Block: minigierka SwingArc, output BasicFirewood
- ✅ Storage system NOWY (2026-04-10): StorageRack/Registry/Manager per species. LogRack=60, FirewoodRack=100, StumpRack=15. 4 aktywne gatunki (Spruce/Birch/Oak/Maple)
- ✅ StorageManager = jedyna authority (2026-04-24): WarehouseManager USUNIĘTY. Sprint C1+C2+C3 — wszystkie callsites zmigrowane. InventoryPanelUI (Tab) pokazuje live dane z SM, zero duplikatów (StorageFamilyHelper.ProductAcceptsSpecies).
- ✅ Firewood unifikacja (2026-04-11): species-agnostic, 3 modele Basic/Fine/Premium
- ✅ FirewoodRack v2 (2026-04-11): U-shape 168v, 3 fill warstwy
- ✅ StumpRack v2 (2026-04-12): per-species bark/cross-section, 10 pniaków (4+3+3), 636v, 1 mat/mesh
- ✅ CarryCrate v2 (2026-04-11): mix crate (List<CrateStack>), koło Q, RackTransferUI, SalesCounterTransferUI, 3 FBX (10/15/25 slotów)
- ✅ EconomyManager, CustomerManager (per species matching)
- ✅ Upgrade System, SawmillManager (hub upgrade'ów, globalne boosty)
- ✅ WorkerManager: NPC pracownicy, 3 niezależne lady, per-rola dictionary
- ✅ TruckManager: 3 TruckData SO (Starter=24/Level2=40/Level3=60)
- ✅ PlantingManager + FertilizeManager + PlantingSlot (3 stage, auto-fertilize, double sapling)
- ✅ Tutorial (33): 9 questów, QuestManager + QuestUI, ISaveable
- ✅ Dialogue System (32): DialogueSO + DialogueUI (typewriter, lokalizacja)
- ⏸️ Dynamic Market: kod gotowy, WYŁĄCZONY — demo statyczne ceny
- ⏸️ Land Expansion (29): odłożone
- ⏸️ NPC Workers + Pathfinding (30): logika gotowa, brak timingu/animacji/ruchu
- ⏸️ Automation (31): odłożone na Early Access

## Faza 4 — UI i polish — KOMPLETNA ✅
- ✅ HUD (34): pieniądze + dzień/czas
- ✅ Notification Queue (35): 3 typy, max 5 stackowanych
- ✅ Tooltip System (36): dwuliniowy, 8 subklas
- ✅ Cursor Management (37): 3 stany, auto-hide
- ✅ Confirmation Dialogs (38): modal Tak/Nie
- ✅ Visual Feedback (39): FloatingTextUI + WorldProgressBarUI
- ✅ Typography/Fonts (40): Nunito, skalowanie
- ✅ Ikony UI (41): 30/30 low-poly 3D (IconRenderer pipeline)
- ✅ Statistics/Journal (42): 11 statystyk + dziennik max 200
- ✅ Ustawienia gracza (43): PauseMenuUI + SettingsUI z 6 zakładkami
- ✅ Accessibility (44): FontSize, HighContrast, 11 skrótów
- ✅ Credits Screen (45): scrollable, 15 języków

## Audio / SFX
- ✅ Ambienty: rzeka, wodospad, las dzień/noc (CC0 Freesound, proximity, crossfade)
- ✅ SFX kroki: 7 powierzchni × 4 warianty (FootstepSystem + SurfaceIdentifier)
- ⬜ SFX drzewa: siekiera, trzask, padanie, uderzenie, kopanie, sadzenie (7)
- ⬜ SFX gracz: skok, podniesienie, odłożenie (3)
- ⬜ SFX rąbanie: idealne/normalne, pudło (3)
- ⬜ SFX pojazd: silnik start/jazda/stop, hamulec, załadunek/rozładunek (6)
- ⬜ SFX sprzedaż: kasa, zamówienie, realizacja, brak pieniędzy (4)
- ⬜ SFX UI: klik, powiadomienie, quest, upgrade, dialog (5)
- ⬜ SFX naprawa: młotek loop, gotowe (2)
- ⬜ SFX magazyn: kłoda na regał, zabranie (2)
- ⬜ Muzyka (Suno) — odłożone

## Środowisko / Polish
- ✅ Day/Night Cycle v2
- ✅ Shadery: VertexColorLit, Waterfall, RockTriplanar, MountainLayered, TerrainTextured, RoadTextured
- ✅ Tekstury: gór, skał, terenu, tartaku, debris, mostu, rampy, lady
- ✅ Korony drzew: vertex color gradient + per-vertex noise (4 gatunki)
- ✅ Góry wewnętrzne v2: 30 gór, 7 profili, 5 warstw shader
- ✅ Światła nocne: 24 lampy (16 street + 8 sawmill), reflektory auta, Forward+ compatible

## Mapa
- ✅ Klif z wodospadem + jaskinia
- ✅ Rzeka semi-eliptyczna + wodospad + tunel
- ✅ Most łukowy
- 🔨 Drogi — 3/6-8 gotowe (brakuje: asfaltowa miasto, ścieżka leśna)
  - ✅ Road_Sawmill_Bridge v2 (2026-04-13): 6m dwupasmowa, face-copy terrainu, linia środkowa
  - ✅ Road_Logging_Dirt + Road_Center_Dirt conformity (2026-04-13): terrain+2cm, 0 pokes, endcap przesunięty żeby nie nachodziły na gravel
  - ✅ Road_Parking_Branch v9 + Parking_Lot (2026-04-14): smooth cubic bezier quarter-arc, shared-vertex junction z Road_Sawmill_Bridge (+4 verts) i Parking_Lot (+5 verts), apron 1m po bokach, 385v/324f. Wyeksportowany do Roads.fbx z Road_Sawmill_Bridge v2, Mat_Road_Gravel + MeshCollider, warstwa Road (layer 6).

## Faza 5 — NPC Customers Traffic (PHASE1_NPC_CUSTOMERS.md)
- ✅ Road layer (2026-04-14): nowa warstwa Road (index 6) w TagManager
- ✅ Infrastruktura waypointów (2026-04-14): TrafficWaypoints z EntryPath (13 main + 5 branch) + ExitPath (5 branch + 13 main) + ParkEntry/ExitYield + 12 ParkingSlots (P00_W..P05_E, 6 par W+E). ParkingManager w Services, Tier1 SO (2 sloty).
- ✅ Parking slots redistribution (2026-04-16)
- ✅ NPC Traffic loop test (2026-04-14)
- ✅ NPCPickup01 visual pipeline (2026-04-16)
- ✅ NPC AI tracking tuning (2026-04-16)
- ✅ Dual-lane reverse parking (2026-04-16)
- ✅ NPC parking exit fix (2026-04-17)
- ✅ ExitPath (2026-04-17)
- ✅ Quest-gated spawn (2026-04-19)
- ✅ NPC pasażer + ciągły spawn (2026-04-19)
- ✅ FinalAlign PD fix (2026-04-19)
- ✅ BranchExitPath manual adjust (2026-04-20)
- ✅ NPCPedestrian + CounterManager + SpawnManager (2026-04-20)
- ⬜ Faza 3: yield wewnętrzny cross-lane

## Sprint Lvl 3 — PlankMaker ✅ COMPLETE (2026-04-30)
## PelletBag / PelletRack ✅ (2026-05-15)
## Shredder / Szreder ✅ (2026-04-20)
## Pelletizer — minigra ✅ WaitForRed phase (2026-05-15)
## Sprint A — Upgrade Shop UI (W TOKU 🔨)
## Sprint C — GrowingTree Persistence ✅ (2026-05-16)

## Drzewa / Assety
- ✅ Spruce, Birch, Oak, Maple: pełny cykl
- ✅ GrowingTreeOak.prefab + GrowingTreeMaple.prefab (2026-05-17)
- ⬜ Acacia, Mahogany — po demo
```

### D:\Unity\Timber_Tycoon\.claude\skills\storage-manager-api\SKILL.md

```md
---
name: storage-manager-api
description: StorageManager API — GetAmount/AddToStorage/RemoveFromStorage + StorageFamilyHelper + ActiveLogSpecies pattern. Używaj gdy piszesz kod czytający lub modyfikujący magazyn.
---

# StorageManager API — Timber Tycoon

Single source of truth for inventory storage. Jedyne API do czytania/pisania stanu magazynu.

## Core API

- `StorageManager.Instance` — singleton, może być null podczas transition/shutdown
- `GetAmount(ProductType product, WoodSpecies species)` → `int`
  - Zwraca sumę ze wszystkich racków dla danej pary (product, species)
  - Dla species-agnostic: zawsze `WoodSpecies.None`
- `AddToStorage(ProductType product, WoodSpecies species, int amount)` → `int`
  - Zwraca OVERFLOW (ile nie zmieściło). Zero = wszystko zmieściło się.
- `RemoveFromStorage(ProductType product, WoodSpecies species, int amount)` → `int`
  - Zwraca ilość FAKTYCZNIE usuniętą. Zero = nic nie było.

## NIE UŻYWAJ — Metody które nie istnieją

- `GetCount()` — NIE ISTNIEJE. Używaj `GetAmount()`
- `GetAllRawMaterials()` — NIE ISTNIEJE (legacy `WarehouseManager`, usunięty Sprint C3)
- `GetAllProducts()` — NIE ISTNIEJE (legacy `WarehouseManager`)
- `HasProduct()` / `HasEnough()` — NIE ISTNIEJE. Użyj `GetAmount() > 0`
- Bezpośredni dostęp do racków z pominięciem StorageManager — **NIGDY**

## Species-agnostic vs species-typed

Przed iteracją gatunków sprawdź: `StorageFamilyHelper.ProductAcceptsSpecies(ProductType pt)`

**Species-agnostic** (`ProductAcceptsSpecies = false`) — czytaj/pisz wyłącznie z `WoodSpecies.None`:
- `Firewood_Regular`, `Firewood_Fine`, `Firewood_Premium`
- `BagOfChips`, `WoodChips`, `Bark`, `Pellet`, `Fertilizer`, `Furniture`

**Species-typed** (`ProductAcceptsSpecies = true`) — każda para (product, species) to osobny bucket:
- `Log`, `Stump`, `Plank`
- Iteruj przez `ActiveLogSpecies[]`, pomijaj `WoodSpecies.None`

## ActiveLogSpecies — wzorzec iteracji

```csharp
private static readonly WoodSpecies[] ActiveLogSpecies =
{
    WoodSpecies.Spruce,
    WoodSpecies.Birch,
    WoodSpecies.Oak,
    WoodSpecies.Maple
};

// Suma wszystkich gatunków
int total = 0;
foreach (var sp in ActiveLogSpecies)
    total += StorageManager.Instance.GetAmount(ProductType.Log, sp);

// Usuń jedną kłodę dowolnego gatunku (LIFO po kolejności w array)
foreach (var sp in ActiveLogSpecies)
{
    if (StorageManager.Instance.GetAmount(ProductType.Log, sp) > 0)
    {
        StorageManager.Instance.RemoveFromStorage(ProductType.Log, sp, 1);
        break;
    }
}
```

Dodanie nowego gatunku (Acacia, Mahogany) = dodaj do `ActiveLogSpecies[]`.
Wzorzec zaimplementowany w: `ChoppingBlock.cs`

## Null safety — obowiązkowy wzorzec

```csharp
if (StorageManager.Instance == null)
{
    Debug.LogWarning("[NazwaKlasy] StorageManager.Instance null — operation skipped");
    return;
}
```

**Każde call site musi mieć null-check.** StorageManager może być null podczas:
- scene transitions
- shutdown
- wczesnej inicjalizacji (przed `Services.Register<StorageManager>()`)

## Pliki — poprawne użycie jako referencja

- `Assets/Project/Scripts/Warehouse/UnloadZone.cs` — `AddToStorage` (pattern rozładunku)
- `Assets/Project/Scripts/Chipper/ChipperMachine.cs` — `GetAmount` + `RemoveFromStorage`
- `Assets/Project/Scripts/ChoppingBlock.cs` — `ActiveLogSpecies` iteration
- `Assets/Project/Scripts/UI/InventoryPanelUI.cs` — `StorageFamilyHelper` branching
```

### D:\Unity\Timber_Tycoon\.claude\skills\timber-delete-safety\SKILL.md

```md
---
name: timber-delete-safety
description: Delete safety — grep project-wide, caller categorization, scene attachment check, compile verify. Używaj przed usunięciem dowolnego pliku .cs, klasy, metody, komponentu lub GameObject.
---

# Delete Safety — Timber Tycoon

Żaden delete nie przechodzi bez wszystkich 4 kroków weryfikacji.

## Zasada fundamentalna

- **Zero delete bez pełnej weryfikacji**
- Delete to akcja jednorazowa — git restore działa, ale nieprzemyślany delete + follow-up commity to godziny debug'u
- Koszt recon'u (5-10 min) << koszt naprawy po zepsutym delete (godziny)
- **Jeśli niepewny kategoryzacji callera → STOP, zapytaj Huntera**

## Obowiązkowy workflow — 4 kroki

Wykonaj WSZYSTKIE 4 kroki w kolejności. Nie skracaj.

### Krok 1 — Grep project-wide

```bash
# Klasa / MonoBehaviour
grep -r "NazwaKlasy" Assets/Project/Scripts/

# Metoda
grep -r "NazwaMetody(" Assets/Project/Scripts/

# Enum value / stała
grep -r "NazwaStałej" Assets/Project/Scripts/
```

Zapisz każde trafienie: plik + linia + kontekst wywołania.

### Krok 2 — Kategoryzacja callerów

Dla każdego trafienia z Kroku 1:

| Kategoria | Definicja | Runtime impact |
|-----------|-----------|---------------|
| **Aktywny** | Wywoływany na głównych ścieżkach gameplay | Blokuje delete |
| **Dead code** | Zero external callerów, nigdy nie wywoływany | Safe |
| **Editor tool** | `[MenuItem]`, `GameObject.Find("...")` — compile-safe | Safe |
| **Komentarz** | Wzmianka w komentarzu / XML docs | Safe |

**Decyzja:**
- Wszystkie trafienia = Dead / Editor / Komentarz → **proceed**
- Jakikolwiek Aktywny → **STOP** — najpierw migracja lub usunięcie callera, potem delete

### Krok 3 — Scene attachment check (MonoBehaviour only)

Dla każdej klasy dziedziczącej `MonoBehaviour` sprawdź attachment w scenie.

Instrukcja dla Huntera:
```
W Unity:
1. Hierarchy search bar → wpisz: t:NazwaKlasy
2. Report: pusto (safe) lub nazwy GameObjects (attached)
```

Jeśli attached → Hunter usuwa component ręcznie → Ctrl+S → dopiero potem delete .cs.

**NIE pomijaj** tego kroku — pominięcie = missing script warning trwale w scenie.
Szczegóły workflow: skill `unity-scene-rules` sekcja "Scene attachment check".

### Krok 4 — Compile check po delete

```bash
git rm NazwaKlasy.cs NazwaKlasy.cs.meta
```

Następnie w Unity (lub przez MCP `check_compile_errors`):
- Zero errors → safe, kontynuuj
- Jakikolwiek error → **`git restore NazwaKlasy.cs`** natychmiast, przeanalizuj przyczynę

## Raport pre-delete — format dla Huntera

Przed zatwierdzeniem, przedstaw tabelę:

| Plik/Klasa | Istnieje? | Ref count | Klasyfikacja | Scene attached? | Decyzja |
|---|---|---|---|---|---|
| `Foo.cs` | ✅ | 3 | 1 active, 2 comment | ⚠️ wymaga check w Unity | STOP |
| `Bar.cs` | ✅ | 0 | dead | brak | proceed |
| `Baz.cs` | ✅ | 2 | 2 editor | brak | proceed |

Tabela musi być kompletna dla wszystkich plików w scope sprint'u.

## Dead code detection — sygnały

- Metody bez external callerów (grep zwraca tylko definicję)
- Klasy z `// TODO: remove in Sprint X` (po zakończonym sprincie)
- Editor tools z hardcoded path'ami do plików które już nie istnieją
- `[ContextMenu]` debug methods bez aktywnego wywołania w kodzie
- Klasy oznaczone `[Obsolete]` które przeżyły migrację
- Scripts z `using` list pustą po usunięciu ciała klasy

## Sprint cleanup pattern

Wzorzec użyty w: B-light (6 dead files), C3 (WarehouseManager), C4-lite (5 legacy files).

1. **Recon** — grep wszystkich plików w scope, raport tabelaryczny
2. **Plan** — kategoryzacja per-plik, decyzje, tabela pre-delete
3. **Hunter approval** — czeka na akceptację planu, NIE proceed bez
4. **Execute** — per-plik: `git rm` → compile check → next
5. **Smoke test** — quick gameplay check po wszystkich delete
6. **Commit** — jeden commit z opisem co i dlaczego

## Rollback

```bash
git status                  # co się zmieniło
git restore NazwaKlasy.cs   # cofnij konkretny plik
git restore .               # cofnij wszystko niestagowane
git reset --hard HEAD~1     # cofnij ostatni commit (NIEODWRACALNE dla niekomitowanych zmian)
```

**`git reset --hard` na już-pushed commit = wymagany force push — NIGDY bez zgody Huntera.**

## Powiązane skills

- `unity-scene-rules` — pełny workflow scene attachment check
- `hunter-communication-style` — reguła "ask before irreversible actions"
```

### D:\Unity\Timber_Tycoon\.claude\skills\timber-migration-pattern\SKILL.md

```md
---
name: timber-migration-pattern
description: Migration pattern Timber Tycoon — recon → plan → implementation → smoke test. Primary + fallback → remove fallback. Używaj gdy migrujesz kod z jednego systemu (np. WarehouseManager) do innego (np. StorageManager).
---

# Migration Pattern — Timber Tycoon

4-fazowy workflow, sprawdzony 4x dzisiaj (UnloadZone, ChipperMachine, ChoppingBlock, InventoryPanelUI). Obowiązkowy dla każdej migracji systemowej.

## Zasada fundamentalna

- Migracja = przeniesienie kodu z **StarySystem** do **NowySystem**
- **Nigdy jednym krokiem.** Zawsze etapy z możliwością rollback'u między nimi
- Pattern "Primary + fallback → remove fallback" = continuous working state przez całą migrację
- Unique log suffix `[#{counter}]` w każdej pętli loggingowej — defeat Unity Console Collapse

## 4-fazowy workflow

### Faza 1 — Recon

Cel: zrozumieć scope przed napisaniem zmian.

1. Grep project-wide wszystkich callerów StarySystem
2. Kategoryzacja każdego call site:
   - **READ** — czyta stan (`GetAmount`, `GetAllRawMaterials`)
   - **WRITE** — modyfikuje stan (`AddRawMaterial`, `RemoveRawMaterial`)
   - **TYPE REF** — tylko deklaracja zmiennej/pola, bez wywołania
3. Sprawdź czy NowySystem ma "secondary mirror" (dual-write) — jeśli tak, Faza 3 = promotion do primary
4. Identyfikuj blockery:
   - API mismatch (StarySystem ma metodę której NowySystem nie ma)
   - Save compatibility (stare save'y tylko w StarySystem)
   - Cross-system readers (inne komponenty czytające StarySystem state)

**Output Fazy 1:** tabelaryczny raport + rekomendacja scope'u dla Huntera.

### Faza 2 — Plan

Cel: konkretny step-by-step plan zanim tknięte linie kodu.

Elementy planu:
- Pliki do modyfikacji (lista z path'ami)
- Per-plik: lista zmian z numerami linii
- Nowe patterns/helpers do dodania (np. `ActiveLogSpecies[]` ze Sprint C2)
- Smoke test po implementacji
- Draft commit message

**Hunter approval OBOWIĄZKOWY przed Fazą 3.** NIE proceed bez.

### Faza 3 — Implementation

Cel: wykonać plan. Bez improwizacji.

Per-plik:
1. Migracja READ paths (`StarySystem.GetX` → `NowySystem.GetY`)
2. Migracja WRITE paths (`StarySystem.AddX` → `NowySystem.AddY`)
3. Null-safety dla `NowySystem.Instance` na każdym call site
4. Unique log suffix `[#{counter}]` dla logów w pętlach
5. Compile check po każdej większej zmianie

Jeśli podczas implementacji pojawi się decyzja nie zawarta w planie → **STOP**, wróć do Fazy 2.

**NIE** modyfikuj kodu poza scope'em planu. Scope creep = rollback risk.

### Faza 4 — Smoke test

Cel: potwierdzenie że migracja nie zepsuła gameplay.

- Compile zero errors
- Zero nowych console warnings (poza planned expected ones)
- **Pełny tutorial regression** — nie tylko unit test migrowanego komponentu
- `InventoryPanelUI` (Tab) pokazuje oczekiwane dane
- Sprzedaż NPC działa end-to-end

**Pełny tutorial jest KRYTYCZNY.** Unit test migrowanego modułu nie wystarczy.

## Primary + Fallback — wzorzec przejściowy

**Etap A — Dual-write (secondary mirror):**
```csharp
// Primary
StarySystem.Instance.AddX(key, 1);
// Secondary mirror — TODO remove after migration
if (NowySystem.Instance != null)
    NowySystem.Instance.AddY(product, species, 1);
```

**Etap B — Promote NowySystem do primary, StarySystem do fallback:**
```csharp
// Primary
NowySystem.Instance.AddY(product, species, 1);
// Fallback (legacy) — TODO remove in Sprint X
if (StarySystem.Instance != null)
    StarySystem.Instance.AddX(key, 1);
```

**Etap C — Remove fallback**
**Etap D — Delete StarySystem.cs** (patrz: `timber-delete-safety`)

## Sprint absorption rule

Jeśli Faza 1 recon odkryje compile blocker w innym planowanym sprincie:
- **Absorb** mniejszy sprint do obecnego
- Udokumentuj w commit message: `"(absorbed Sprint X: reason)"`

## Unique log suffix — OBOWIĄZKOWY dla pętli

```csharp
// ❌ ZŁE
Debug.Log($"[UnloadZone] Unloaded 1x {pt} ({sp}) to StorageManager");

// ✅ DOBRE
Debug.Log($"[UnloadZone] Unloaded 1x {pt} ({sp}) to StorageManager [#{++unloadCount}]");
```

## Historia migracji — referencje

| Sprint | StarySystem | NowySystem | Pattern |
|--------|-------------|------------|---------|
| C1 | `WarehouseManager` (ChipperMachine) | `StorageManager` | full cut |
| C2 | `WarehouseManager` (ChoppingBlock) | `StorageManager` + `ActiveLogSpecies[]` | full cut |
| C3 | `WarehouseManager` (UnloadZone) | `StorageManager` | remove fallback |
| C3* | `WarehouseManager` (InventoryPanelUI) | `StorageManager` + `StorageFamilyHelper` | absorbed z C5 |

## Powiązane skills

- `timber-delete-safety` — Etap D workflow
- `unity-scene-rules` — jeśli migracja dotyka scene-attached components
- `storage-manager-api` — API cheat sheet dla StorageManager
- `hunter-communication-style` — Hunter approval obowiązkowy przed Fazą 3
```

### D:\Unity\Timber_Tycoon\.claude\skills\unity-scene-rules\SKILL.md

```md
---
name: unity-scene-rules
description: Unity scene files Timber Tycoon — aktywna scena, format binary vs YAML, missing script handling, scene attachment check. Używaj gdy modyfikujesz lub analizujesz scenę.
---

# Unity Scene Rules — Timber Tycoon

Jedna aktywna scena, format binarny — edycja tylko przez Unity Editor.

## Aktywna scena

- **Ścieżka:** `Assets/Demo_Scene.unity`
- **Format:** BINARY (Unity 6000.3.5f1 native serialization)
- **NIE edytuj tekstem** — str_replace, sed, Write tool spowodują corruption sceny
- Modyfikacje: WYŁĄCZNIE manualnie w Unity Editor przez Huntera
- **Nie ma innych scen w projekcie** — `Assets/Scenes/` folder usunięty (legacy Phase 1 duplikat).

## Format — binary vs YAML

- **Domyślnie Unity 6: BINARY** (aktywna scena TT jest binarna)
- YAML scenes: tylko przy `ProjectSettings/EditorSettings Force Text`
- Binary: pierwsze bajty nie-ASCII
- YAML: zaczyna się od `%YAML 1.1` lub `--- !u!`
- Prefabs (`.prefab`): zawsze YAML

## Modyfikacje sceny — workflow

1. Napisz instrukcję dla Huntera:
```
Akcja w Unity (~X min):
1. Hierarchy → [konkretny GameObject]
2. [akcja]
3. Ctrl+S
```
2. **POCZEKAJ na potwierdzenie**
3. Commituj po potwierdzeniu

**NIGDY:**
- Nie edytuj `Assets/Demo_Scene.unity` tekstowo
- Nie usuwaj pliku sceny bez explicit request Huntera

## Missing script warnings

Naprawa (Hunter manual):
1. Otwórz Unity → `Assets/Demo_Scene.unity`
2. Hierarchy search → żółty trójkąt
3. Inspector → prawy klik `Missing (Mono Script)` → Remove Component
4. Ctrl+S → commit

## Scene attachment check — przed każdym delete .cs

1. Instrukcja dla Huntera:
```
W Unity:
1. Hierarchy search bar → wpisz: t:NazwaKlasy
2. Report: pusto (safe) lub nazwy GameObjects (attached)
```
2. Jeśli attached → Hunter usuwa component → Ctrl+S
3. Dopiero potem: `git rm NazwaKlasy.cs NazwaKlasy.cs.meta`

## Konwencje

- Play Mode: **NIGDY `save_scene` ani `DestroyImmediate` podczas Play Mode**
- Scene merge conflicts: wymagają Unity SmartMerge lub manual rework
```

## Section 8: Junction & Git verification

### dir D:\Unity\Timber_Tycoon\kb

```
    Directory: D:\Unity\Timber_Tycoon\kb

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        17.05.2026     10:56                engine
d-----        17.05.2026     10:56                genre
d-----        17.05.2026     10:57                projects
d-----        17.05.2026     10:57                templates
d-----        17.05.2026     10:56                workflow
d-----        17.05.2026     10:56                _archive
d-----        17.05.2026     10:57                _inbox
-a----        17.05.2026     10:57             87 .gitignore
-a----        17.05.2026     10:56           1043 CLAUDE.md
-a----        17.05.2026     10:56           1931 MOC.md
-a----        17.05.2026     10:56           1716 README.md
```

### git log --oneline -10 (D:\Hunter\KnowledgeBase)

```
f66cdb9 Initial KB structure
```

### git status (D:\Hunter\KnowledgeBase)

```
On branch master
nothing to commit, working tree clean
```

### git remote -v (D:\Hunter\KnowledgeBase)

```
(no output — no remote configured)
```

## Section 9: Toolchain inventory

### claude mcp list

```
Checking MCP server health…

claude.ai Google Drive: https://drivemcp.googleapis.com/mcp/v1 - ✓ Connected
claude.ai Gmail: https://gmailmcp.googleapis.com/mcp/v1 - ! Needs authentication
claude.ai Google Calendar: https://calendarmcp.googleapis.com/mcp/v1 - ! Needs authentication
blender-mcp: uvx blender-mcp - ✓ Connected
elevenlabs: uvx elevenlabs-mcp - ✓ Connected
coplay-mcp: uvx --python >=3.11 coplay-mcp-server@latest - ✓ Connected
```

### C:\Users\<user>\.claude.json

EXISTS: 34711 bytes

### D:\Unity\Timber_Tycoon\.claude.json

MISSING

## Section 10: Hooks (if configured)

### C:\Users\<user>\.claude\hooks\

FOLDER_MISSING

### D:\Unity\Timber_Tycoon\.claude\hooks\notify-input-needed.cmd

```cmd
@echo off
REM Notification hook - alerts Hunter when Claude Code needs input
REM Uses PowerShell toast notification on Windows

powershell -NoProfile -Command "[void][System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); $balloon = New-Object System.Windows.Forms.NotifyIcon; $balloon.Icon = [System.Drawing.SystemIcons]::Information; $balloon.BalloonTipTitle = 'Claude Code'; $balloon.BalloonTipText = 'Czekam na Twoja odpowiedz!'; $balloon.Visible = $true; $balloon.ShowBalloonTip(5000); Start-Sleep -Seconds 6; $balloon.Dispose()"
```

### D:\Unity\Timber_Tycoon\.claude\hooks\post-compact-context.cmd

```cmd
@echo off
REM SessionStart hook (after compact) - re-injects critical project context

echo CRITICAL REMINDERS (post-compact):
echo ============================================
echo WORKFLOW: Zawsze klasyfikuj zadanie do Poziomu 1/2/3 PRZED implementacja.
echo Gdy niepewny -- wybierz wyzszy (bezpieczniejszy) poziom.
echo --------------------------------------------
echo - NIGDY nie zapisuj sceny w Play Mode
echo - NIGDY nie uruchamiaj editorowych skryptow w Play Mode
echo - ZAWSZE rob backup sceny PRZED modyfikacjami
echo - Forward auta = -transform.right (FBX quirk)
echo - Gracz NIE ma ekwipunku -- magazyn jest jedynym skladowiskiem
echo - Klody NIGDY nie sa sprzedawane -- zawsze przetwarzane
echo - Proceduralne tekstury Blendera MUSZA byc baked do PNG
echo - Performance: max 2M verts, 500 draw calls
echo - Sawmill pozycja (177.9, 7.62, -88.71) -- NIE ZMIENIAC
echo --------------------------------------------
echo PIERWSZA AKCJA: przeczytaj .claude/checkpoint.md jesli istnieje
echo Po skonczonym zadaniu: zasugeruj /done gdy Hunter wyrazi satysfakcje
echo ============================================
```

--- INTAKE COMPLETE ---
