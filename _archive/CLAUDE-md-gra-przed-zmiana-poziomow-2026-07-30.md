# Kerf - Sawmill Tycoon — Kontekst Projektu

## Przegląd
Single-player FPP tycoon o tartaku. Gracz wycina drzewa, przetwarza drewno, sprzedaje produkty, rozbudowuje tartak.
- **Unity 6000.5.1f1**, URP 17.5, New Input System (wersja z Packages/manifest.json)
- **Pipeline**: Hunter (reżyser/decyzje) → Claude Code (implementacja) → MCP (Coplay, Blender, ElevenLabs)
- **Plan**: `Docs/TIMBER_TYCOON_FINAL_PLAN.md` — jedyne źródło prawdy o scope
- **Styl rozmowy z Hunterem**: @.claude/skills/hunter-communication-style/SKILL.md
- **Aktywne lekcje (auto)**: @.claude/skills/active-rules/SKILL.md
- **Unity API**: ładowany jako plugin — `claude --plugin-dir ./unity-claude-skills`

## Hunter — kontekst użytkownika

Hunter nie programuje i nie tworzy gier technicznie. Jest creative directorem — decyduje CO ma być zrobione, WIZUALNIE i FUNKCJONALNIE. Implementację technicznie deleguje do Claude Code.

Konsekwencje:
- Wyjaśniaj prostym językiem — bez żargonu programistycznego. Nazwy systemów zaimplementowanych w projekcie (`ServiceLocator`, `GameEventSO`) możesz używać, ale zawsze PO polskim wyjaśnieniu i w nawiasie (patrz hunter-communication-style, Zasada nr 2-3), nigdy zamiast wyjaśnienia.
- Nie zakładaj, że Hunter rozpozna bug w kodzie — TY masz go znaleźć i opisać
- Przy propozycjach technicznych — zawsze tłumacz DLACZEGO, nie tylko CO
- Hunter ocenia wyniki na podstawie efektu wizualnego/gameplay, nie elegancji kodu

Pełny protokół współpracy: @.claude/skills/hunter-communication-style/SKILL.md

---

## Workflow obowiązkowy — 3 poziomy analizy przed implementacją

**Każde zadanie od Huntera klasyfikuj do jednego z 3 poziomów PRZED rozpoczęciem pracy. Jeśli niepewny, który poziom pasuje → wybierz poziom z większą analizą i akceptacją, czyli ten o niższym numerze (Poziom 1 jest najbezpieczniejszy, Poziom 3 najmniej).**

### Poziom 1 — PEŁNA ANALIZA + AKCEPT (obowiązkowa)

**Kiedy stosuj:**
- Refaktor istniejącego systemu
- Zmiany edytujące (zapis/usunięcie) treść 3+ plików — automatycznie generowane pliki .meta i pliki tylko czytane bez edycji się nie liczą
- Zmiany architektoniczne (wprowadzenie nowego wzorca projektowego, nowy manager, albo zmiana sposobu działania istniejącego wzorca projektowego — np. ServiceLocator, GameEventSO, ISaveable, Singleton)
- Dotknięcie hardcoded constraints: pozycja tartaku (177.9, 7.62, -88.71), klif, wodospad, performance budget, forward axis auta (-transform.right), scale inconsistency Spruce_Log/Birch_Log (znana, ale tu nieudokumentowana rozbieżność skali między tymi dwoma logami — przed jakąkolwiek zmianą ich skali dopytaj Huntera lub sprawdź szczegóły, NIE zmieniaj „na czuja")
- Zmiany dotykające zapisu (ISaveable) — migracja save'ów
- Zadania w Plan Mode (Shift+Tab)
- Pytania informacyjne: "co myślisz", "jak byś to zrobił", "co polecasz", "jak najlepiej", "mam dylemat", "nie wiem czy"

**Co robisz:**
1. **Analiza** (co widzisz jako problem/zadanie, jaki jest kontekst w grze, jakie systemy się łączą)
2. **Opcje** (2-3 możliwe rozwiązania z pros/cons każdego w kontekście Kerf - Sawmill Tycoon)
3. **Rekomendacja** (które rozwiązanie wybierasz i dlaczego — połącz z istniejącą architekturą)
4. **Pytanie akceptacyjne** ("Czy akceptujesz rozwiązanie X, czy chcesz żebym wrócił do Y/Z?")
5. **STOP** — czekaj na akcept Huntera. NIE implementuj.
6. Jeśli Hunter zgłosi uwagi → ponów krok 1-4 z uwzględnieniem jego uwag, przedstaw za/przeciw jego sugestii, zarekomenduj ulepszone rozwiązanie.
7. Dopiero po jednoznacznej zgodzie na implementację ("ok/tak/zatwierdzam/rób") → implementuj. Te same słowa-zgody obowiązują w Poziomie 1 i Poziomie 2. Słowo "ok" powiedziane W ODPOWIEDZI NA PROPOZYCJĘ (przed rozpoczęciem pracy) oznacza zgodę na start implementacji, NIE sygnał /done (/done dotyczy satysfakcji z gotowego wyniku — patrz "Kiedy sugerować /done").

### Poziom 2 — KRÓTKIE UZASADNIENIE + AKCEPT (orientacyjnie 1-2 zdania, nie twardy limit)

**Kiedy stosuj:**
- Nowy plik kodu / feature addytywny, który wpisuje się w istniejący wzorzec BEZ tworzenia nowego managera ani nowego wzorca (np. kolejny komponent ISaveable obok istniejących, lub nowy typ drzewa odwzorowany 1:1 na wzór Spruce/Birch/Oak/Maple). Jeśli plik/feature wprowadza nowy manager lub nowy wzorzec → Poziom 1.
- Zmiana dwóch lub więcej wartości w jednym ScriptableObject
- Nowy prefab według ustalonego pipeline (model → bake → FBX → Unity)

**Co robisz:**
1. **Krótkie uzasadnienie** (orientacyjnie 1-2 zdania, dlaczego to rozwiązanie i jak pasuje do projektu — to wskazówka długości, nie sztywny limit; nie tnij treści na siłę, by się zmieścić)
2. **Pytanie akceptacyjne** ("Potwierdzasz?")
3. Po akcept → implementuj.

### Poziom 3 — OD RAZU IMPLEMENTACJA (bez analizy)

**Kiedy stosuj:**
- Zmiana DOKŁADNIE jednej wartości w istniejącym SO (color, speed, capacity, amount; 2+ wartości = Poziom 2)
- Przemianowanie zmiennej/pliku
- Usuwanie pojedynczego komponentu z obiektu — najpierw zastosuj skill `timber-delete-safety` (jak nakazuje sekcja „Ładowanie skills"), a po jego sprawdzeniach implementuj.
- Literówki, formatowanie, komentarze w kodzie
- Przesunięcie obiektu na scenie o zadaną wartość (bez zmiany architektury) — Z WYJĄTKIEM obiektów objętych hardcoded constraints z Poziomu 1 (m.in. tartak, klif, wodospad), które zawsze są Poziomem 1, nawet przy zwykłym przesunięciu.

**Co robisz:**
Implementuj od razu. Po zakończeniu — krótki raport co zostało zmienione.

---

## Zasady — BEZWZGLĘDNE (nigdy nie naruszaj niezależnie od poziomu)

- **ZAWSZE backup sceny** PRZED modyfikacjami scen
- **NIGDY save_scene w Play Mode** — niszczy obiekty permanentnie
- **NIGDY editorowe skrypty w Play Mode** — DestroyImmediate = utrata danych
- **NIGDY nie oddawaj Hunterowi pracy do playtestu bez sprawdzenia w BUILDZIE** — patrz sekcja „Weryfikacja w buildzie" niżej. „Działa w Edytorze" nie jest twierdzeniem o grze, tylko o Edytorze.
- **NIGDY nie implementuj nowego feature'a ani zmiany scope'u spoza planu** (`Docs/TIMBER_TYCOON_FINAL_PLAN.md` — jedyne źródło prawdy o scope) bez potwierdzenia Huntera. Drobne czynności Poziomu 3 (literówki, formatowanie, zmiana jednej wartości w istniejącym SO, przemianowanie) nie są tym objęte.
- **Gameplay values ZAWSZE w ScriptableObjects** — zero magic numbers w kodzie
- **Performance**: max 2M verts i 500 draw calls na jeden renderowany kadr (to, co widzi kamera); cel: 60 FPS minimum na GTX 1050 / RX 560 lub lepszych.
- **Git**: conventional commits, LFS dla binarnych

---

## Weryfikacja w buildzie (od 2026-07-13)

Gra przez rok nie była budowana i uzbierała całą klasę uśpionych błędów: **kod, który w Edytorze działa,
a w buildzie pada albo cicho umiera.** Trzy blokery wyszły w ciągu godziny od pierwszego builda
(kolizje drzewek, crash startu sceny, player się nie kompilował), a czwarty (magentowe znaczniki dostaw
+ martwe podświetlanie questów) — dopiero gdy Hunter w tym buildzie zagrał.

**Edytora NIE DA SIĘ oszukać, żeby udawał build.** Nie szukaj obejść — zbuduj.

### Bramka nr 1 (twarda): przed KAŻDYM oddaniem pracy Hunterowi

Zanim powiesz „gotowe / do playtestu / `/done`" — **build + sonda muszą przejść**:

```powershell
# Unity MUSI byc zamkniete
& "D:\Unity\6000.5.1f1\Editor\Unity.exe" -batchmode -quit -projectPath "D:\Unity\Timber_Tycoon" `
   -executeMethod BuildTimberTycoon.BuildWindows -logFile "_Handoff\build_game.log"

.\Builds\Win64\Kerf.exe -meshaudit -logFile "_Handoff\player.log"
echo $LASTEXITCODE          # 0 = OK, 1 = jest FAIL
type .\Builds\Win64\MeshAudit_Result.txt
```

Sonda (`MeshReadabilityProbe`, `Assets/Project/Scripts/Testing/`) sprawdza w PRAWDZIWYM buildzie:
collidery tworzone w runtime (drzewka, kępki ziemi), integralność sceny (martwe komponenty = crash startu),
klamkę lakierni, oraz **magentę** (materiały bez shadera + czy shadery szukane przez kod są w buildzie).

### Bramka nr 2: build NATYCHMIAST, nie czekając na koniec

Gdy zmiana dotyka rodziny **„Edytor kłamie"** — buduj od razu, nie zbiorczo:

- materiały tworzone w kodzie (`new Material`, `Shader.Find`, `GameObject.CreatePrimitive`)
- collidery i odczyt siatek w trakcie gry (`AddComponent<MeshCollider>`, `mesh.vertices/triangles`,
  `hit.triangleIndex`, `hit.textureCoord`) → wymagają Read/Write Enabled na modelu
- komponenty dokładane skryptami setupowymi (`AddComponent<T>` w edytorze) → klasa MonoBehaviour
  **MUSI** leżeć w pliku o swojej nazwie, inaczej scena nie wczyta się w buildzie
- API edytora (`AssetDatabase`, `UnityEditor.*`) w kodzie runtime → player się nie skompiluje
- każda zmiana sceny

Reszta zmian (wartości w SO, teksty, balans, UI) — **zbiorczo**, przy bramce nr 1.

### Bramka nr 3: sonda rośnie

**Każda naprawa z tej rodziny dopisuje do sondy sprawdzenie**, które by ją wyłapało. Sonda to
build-smoke projektu — ma pilnować przeszłych błędów, żeby nie wracały.

Pełny raport i tabela: `_Handoff/MESH_READABLE_AUDIT.md`.

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

## Struktura kodu (lista folderów — opisy niepełne)
```
Assets/Project/Scripts/
├── Chipper/
├── Core/          — ServiceLocator, GameEventSO, GameState, ISaveable, TreeTypeData, Interactable, QuestManager
├── Debug/
├── Editor/
├── Inventory/     — InventorySystem (rejestr magazynowy), StorageZone
├── Items/         — CarryCrate, SalesCounter, CounterRepair, RecipeData
├── Machines/      — MachineBase (abstract), MachineController, MachineOutputCalculator, MachineData, MachineDatabase, MachineRecipe, MachineTier, MachineInteraction, Pelletizer/ (PelletizerMachine, DieAssemblyController)
├── Managers/      — SaveManager, AudioManager, TimeManager, InputManager, SceneTransitionManager, LocalizationManager, EconomyManager, CustomerManager, DynamicMarketManager, UpgradeManager (WorkerManager→NPC/Workers/, TruckManager→Vehicle/, SawmillManager→Upgrades/, PlantingManager+FertilizeManager→Planting/)
├── NPC/
├── PlankMaker/
├── Planting/
├── Player/        — PlayerController, PlayerInteraction
├── Products/
├── Storage/       — StorageManager, StorageRackRegistry, StorageFamily, RackStack, CrateManager, LoadingStation, WoodSpecies
├── UI/            — HUD, dialogi, minigry, ustawienia, InventoryPanelUI
├── Upgrades/
├── Vehicle/       — VehicleController, VehicleCamera, VehicleStorage, VehicleHUD
├── Warehouse/     — UnloadZone (uwaga: StorageRack.cs leży w katalogu głównym Scripts/, nie tutaj; klasy WarehouseManager już nie ma)
└── Wood/
```

## Konwencje

### Kod
- **Brak namespace'ów** — flat global namespace
- **4 spacje**, braces Allman style
- Klasy/metody: `PascalCase`, pola: `camelCase`
- Wzorce: ServiceLocator + GameEventSO + ISaveable + Singleton (równolegle)
- SO: `[CreateAssetMenu(menuName = "Kerf/...")]`

### Assety
- Materiały: `Mat_NazwaOpisu.mat`
- Modele: `NazwaObiektu.fbx`
- Tekstury: `NazwaObiektu_Bake.png` (proceduralne tekstury Blendera NIE eksportują się z FBX — bake do PNG)
- Prefaby: `NazwaObiektu.prefab`
- SO Items: `Item_Typ_Wariant.asset`
- Events: `OnNazwaEventu.asset` w ScriptableObjects/Events/

## Kluczowe decyzje designerskie (nie ruszaj bez rozmowy z Hunterem)
- Single-player, mapa gotowa (brak proc-gen)
- Narzędzia niezniszczalne, tempo gry jednostajne (gra nie przyspiesza ani nie zwalnia z postępem — bez rampy tempa).
- Gracz NIE ma ekwipunku — InventorySystem = rejestr magazynowy
- BEZ: pogody/sezonów, minimapy, random events, difficulty, wildlife
- VFX minimalne - dozwolone TYLKO te: dym, iskry, splash, monety, mgiełka wodna, kurz, para, trociny/wióry (wyłącznie przy cięciu piłą taśmową), mgiełka lakieru (WYŁĄCZNIE z dyszy pistoletu w fazie lakierowania w stolarni; zgoda Huntera 2026-07-21) - lista zamknięta; poszerzona 2026-07-18 i 2026-07-21 decyzjami Huntera. Wszystkie inne efekty (liście, błyski, fala uderzeniowa itp.) - NIE. Rozmieszczenie po playtestach 18.07: wodospad (mgiełka+splash), rębak (kurz pracy), kompostownik (dym TYLKO w minigrze: szary w zieleni, czarny przy przegrzaniu, gaśnie poniżej), piła i stolarnia (DROBNE trociny w punkcie styku ostrza, wspólny emiter SawdustVFX), upadek drzewa (kurz wzdłuż CAŁEGO pnia). BEZ cząsteczek: pieniek, peleciarka.
- Meble odłożone — typy do ustalenia z Hunterem

---

## Higiena kontekstu

### Kiedy sugerować `/done`
Sugeruj `/done` tylko gdy fraza wyraża satysfakcję z UKOŃCZONEGO wyniku zadania — rozpoznaj po frazach typu: "super", "dzięki", "działa", "ok", "zatwierdzam", "gotowe", "idealnie", "dobra", "perfect". Jeśli w tej samej wiadomości pojawia się kolejne polecenie lub poprawka (np. "ok, a teraz zmień X"), traktuj to jako zwykłe potwierdzenie w trakcie pracy, nie sygnał `/done`.

NIE sugeruj `/done`:
- Po każdej odpowiedzi automatycznie
- Gdy Hunter wciąż zgłasza poprawki lub iteruje
- Gdy jesteś w środku wieloetapowego zadania
- Gdy fraza satysfakcji ("ok", "dobra") pojawia się w tej samej wiadomości razem z nową prośbą o poprawkę (np. "ok, teraz popraw X") — to potwierdzenie w trakcie iteracji, nie sygnał `/done`

### Kiedy sugerować `/clear` vs `/compact`
- **Przełączenie na inny system projektu** (np. z drzew na UI) → sugeruj `/clear`
- **Kontynuacja tego samego systemu z nowym zadaniem** → sugeruj `/compact Focus on <nazwa systemu>`
- **Po auto-compact** → PIERWSZA akcja: przeczytaj `.claude/checkpoint.md` jeśli istnieje

### Długa sesja z powtórzeniami — kiedy reagować
Gdy w obecnej sesji zauważysz:
- **5+ razy wracałeś do tego samego pliku/systemu z kolejnymi poprawkami** (licz całą bieżącą sesję, także pracę sprzed auto-compact — nie zeruj licznika po skróceniu historii), LUB
- **3+ razy cofałeś własne zmiany** w bieżącej sesji — gdzie jeden cykl „dodałeś coś → Hunter poprosił o usunięcie → potem o powrót" liczy się jako jedno cofnięcie

Powiedz Hunterowi wprost: "Ta sesja jest już obciążona wieloma iteracjami nad [nazwa systemu]. Kontekst ma dużo szumu ze starych wersji. Zalecam `/checkpoint` + `/compact Focus on [aktualny stan systemu]` przed kolejną zmianą, żeby odciążyć kontekst."

### Ładowanie skills
Ładuj skill TYLKO gdy bieżące zadanie bezpośrednio dotyczy jego tematu (WYJĄTEK: `active-rules` i `hunter-communication-style` — importowane przez @ w sekcji Przegląd, oznaczone jako "auto" / "Apply on EVERY task" — obowiązują zawsze, niezależnie od tematu):

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

Gdy niepewny czy ładować konkretny skill — NIE ładuj i poproś Huntera o potwierdzenie. (Po akcepcie zadania, w trakcie implementacji, ma pierwszeństwo sekcja „Automatyzacja" — wtedy sam wybierasz skills bez pytania.)

---

## Delegowanie do agentów

- **3D modele** (drzewa, budynki, pojazdy, narzędzia, props) → agent `blender-modeler`
- **Teren / mapa / góry / rzeki / drogi** → agent `blender-terrain`
- **Operacje sceny Unity przez Coplay MCP** → agent `unity-operator`

Formuła delegacji: użyj Task tool (narzędzie uruchamiające subagenta) z promptem zaczynającym od "Use a subagent to...". NIE wykonuj w głównej sesji zadań przypisanych tym agentom (tworzenie/edycja modeli 3D, teren/mapa, oraz operacje budujące/przebudowujące scenę przez Coplay MCP) — zawsze deleguj. Wyjątkiem są pojedyncze drobne operacje sceny dozwolone w Poziomie 3 (np. przesunięcie jednego obiektu o zadaną wartość), które wykonujesz sam.

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
- Nie przerywaj w środku pełnej implementacji, żeby dopytać o szczegóły techniczne — wyjątkiem są sprzeczne wymagania lub niejednoznaczność, która może popsuć system; wtedy przerwij i zapytaj. Niezależnie od tego, zasady BEZWZGLĘDNE (sekcja wyżej) zawsze obowiązują — wymagane przez nie potwierdzenia (np. implementacja spoza planu) nie są „przerywaniem", którego zakazuje ta reguła.
