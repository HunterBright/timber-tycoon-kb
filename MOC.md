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

## Silnik - lekcje (158)

Bledy rozwiazane i pulapki zlapane w Unity i Blenderze.

- [[20260722-2055-raycast-w-gore-nie-widzi-tafli-wody|"Czy jestem pod wodą?" - promień w GÓRĘ nic nie zobaczy (backface)]] 
- [[urp-distant-caster-shadow-band|"Dark band that follows the player" = terrain self-shadow leaking onto near-coplanar road meshes]] [OK]
- [[20260727-2140-linijka-wysokosci-na-zdjeciu-jest-krzywa|"Ile procent kadru, tyle procent wysokości" to nieprawda, gdy obiektyw patrzy z góry]] 
- [[20260610-1345-unity-instance-setup-divergence|"Works for product A, dead for product B" = per-instance setup divergence, not code]] 
- [[20260704-1556-data-complete-not-reachable|"Wszystkie assety gotowe" ≠ grywalne - zawsze prześledź ŚCIEŻKĘ ZDOBYCIA, nie tylko obecność danych]] 
- [[20260720-0940-dwa-niezalezne-bledy-gamma-w-jednym-modelu|"Wszystko za jasne" po podmianie modelu: dwa niezależne błędy gamma dające ten sam objaw]] 
- [[20260726-1810-ciagla-powloka-zamiast-osobnych-bryl|"Zle przyklejone konczyny" to nie blad ustawienia, tylko blad architektury]] 
- [[20260616-1532-unity-9slice-contentsizefitter-pollution|9-slice Image na obiekcie layoutu zaniża ContentSizeFitter (panel rośnie do sumy borderów)]] 
- [[20260613-0625-9slice-ppu-must-scale-to-target-rect-not-stay-100|A large 9-slice sprite at PixelsPerUnit=100 breaks because its fixed corners exceed the panel]] 
- [[20260615-1030-service-locator-optional-vs-required-lookup|A service locator's Get() must distinguish OPTIONAL from REQUIRED lookups, or it false-alarms at start/teardown]] 
- [[20260629-1916-unity-minigame-abort-cleanup|Aborting a coroutine-driven minigame: release the active-flag LAST, and StopAllCoroutines]] 
- [[20260722-1130-canvas-addcomponent-kills-buildui-silently|AddComponent<Canvas> dokłada RectTransform - kolejny AddComponent<RectTransform> rzuca wyjątkiem i po cichu urywa budowę UI w Awake]] 
- [[20260623-0855-unity-layoutelement-requirecomponent-recttransform-null-trap|AddComponent<RectTransform>() zwraca null po wcześniejszym AddComponent komponentu z [RequireComponent(RectTransform)]]] 
- [[20260710-2110-unity-addobjecttoasset-saveassets-order|AddObjectToAsset wymaga SaveAssets() PRZED ImportAsset(), inaczej sub-asset znika]] 
- [[20260626-0929-unity-major-upgrade-deprecation-cleanup|Aktualizacja major Unity → wyjście z Safe Mode + sprzątanie deprecacji]] 
- [[20260613-1200-unity-offscreen-render-rig-scene-leaks|An "isolated" offscreen render rig still inherits the open scene's lights AND skybox reflection]] 
- [[20260724-0907-arcade-car-climbs-walls|Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo]] 
- [[bake-space-transform-linked-duplicates-rotation-bug|bake_space_transform + Linked Duplicates = 90° Rotation Injection]] [szkic]
- [[20260618-0724-blender-ortho-ui-sprite-bake-framing|Baking flat UI sprites in Blender: ortho frame width = ortho_scale × 2]] 
- [[20260721-0725-frame-time-epsilon-guard-breaks-at-high-fps|Bezpiecznik `Time.deltaTime > 0.001f` cicho zeruje sygnał, gdy build chodzi szybciej niż 1000 FPS]] 
- [[20260612-0950-blender51-builtin-mcp-protocol|Blender 5.1 ma wbudowane rozszerzenie MCP - niekompatybilne ze starym blender-mcp]] 
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] 
- [[20260705-0840-blender-linear-color-into-unity-srgb-material|Blender LINIOWE Base Color wpisane wprost w Unity Color property = ~1 gamma za ciemno (projekt Linear)]] 
- [[20260725-1930-blender-pixels-buffer-not-converted-to-srgb-on-png-save|Blender nie przelicza `image.pixels[]` na sRGB przy zapisie PNG]] 
- [[20260704-1322-blender-file-image-scale-bake-revert|Blender: image.scale() na file-backed image nie trzyma się podczas bake - użyj images.new()]] 
- [[20260722-1625-measure-before-fixing-serialization-hunch|Brakujący klucz w assecie NIE oznacza zera - zmierz, zanim "naprawisz"]] 
- [[capsule-collider-direction-axis|CapsuleCollider Direction Axis Cheatsheet]] [szkic]
- [[20260716-1812-charactercontroller-depenetration-thin-mesh-terrain|CharacterController + cienki jednostronny teren-siatka = gracz pod mapą (i jak się przed tym bronić)]] 
- [[20260625-0712-charactercontroller-velocity-freezes-footsteps|CharacterController.velocity „zamraża się" gdy przestajesz wołać Move() → audio sterowane ruchem przecieka]] 
- [[20260714-2350-null-physics-material-silently-becomes-default-friction|Collider z materiałem `null` NIE ma zerowego tarcia - ma DOMYŚLNE 0.6]] 
- [[20260628-1140-conform-road-mesh-to-edited-terrain|Conforming an existing road/decal mesh to terrain that was edited later]] 
- [[20260713-1420-convex-meshcollider-swallows-hollow-interiors|Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku]] 
- [[20260609-1045-coplay-execute-script-roslyn-diagnostic-crash|Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)]] 
- [[20260710-2252-coplay-execute-script-tmpro-compile-fail|Coplay execute_script nie kompiluje plików z `using TMPro;`]] 
- [[20260721-1340-stale-artifact-from-crashed-test-run|Crashed test run leaves the PREVIOUS report on disk and reads as "my code never got built"]] 
- [[20260531-1530-unity-humanoid-autorig-mirrored-foot|Crooked foot under Unity Humanoid = auto-rig copied the foot bind pose instead of mirroring it]] 
- [[cylindric-beams-visual-contrast|Cylindric vs Rectangular Beams for Visual Contrast]] [szkic]
- [[20260718-0800-particle-visibility-water-sorting|Czasteczki "dzialaja, ale ich nie widac" - trzy niezalezne przyczyny przy wodzie]] 
- [[debugging-search-first-trust-render-check-upstream|Debugging methodology: search-first, trust the render, check for upstream sabotage]] [OK]
- [[20260707-0715-navmesh-decorative-collider-carves-service-line|Dekoracyjny prop z colliderem na linii chodzenia wycina dziure w NavMesh i wypycha NPC do srodka budynku]] 
- [[desaturated-colors-for-low-poly|Desaturated Colors for Low-Poly Aesthetic]] [szkic]
- [[20260628-1615-footstep-surface-raycast-from-feet|Detekcja nawierzchni pod graczem: strzelaj raycastem OD STÓP, nie od środka postaci]] 
- [[20260531-0934-humanoid-orientation-from-armature-not-bbox|Determine a humanoid's up/forward axis from ARMATURE bone landmarks, not from bounding-box max-spread - a T-pose arm span can beat true height]] 
- [[20260702-2140-shader-property-stale-serialized-material-values|Dodanie property do shadera może aktywować STARE, ukryte wartości w materiałach]] 
- [[dynamic-rigidbody-no-nonconvex-meshcollider|Dynamic Rigidbody → Primitive or Convex Collider, Never Non-Convex Mesh]] [szkic]
- [[20260625-1135-unity-3d-sound-quiet-listener-distance|Dźwięk 3D jest za cichy nie przez głośność, tylko przez odległość kamery (AudioListener) od źródła]] 
- [[20260626-1016-unity-one-sided-audio-channel-balance|Dźwięk słychać tylko z jednej strony → najpierw sprawdź balans kanałów ŹRÓDŁA]] 
- [[20260628-1702-editmode-collider-cook-scatter-mask|Edit-mode placement that excludes geometry via physics fails for non-convex runtime/embedded MeshColliders]] 
- [[editor-scene-view-input-capture|Editor Scene View Input Capture Pattern]] [szkic]
- [[20260611-2055-editor-playmode-test-harness-quirks|Editor-driven Play Mode test automation - three engine quirks that break naive harnesses]] 
- [[20260702-1955-playmode-script-edit-domain-reload|Edycja skryptów w trakcie Play Mode zabija statyczne rejestry (domain reload w locie)]] 
- [[20260725-2320-fartuch-skinning-srednia-dwoch-ud-daje-zero|Fartuch ważony po połowie na oba uda NIE RUSZA SIĘ przy chodzie]] 
- [[fbx-binary-overwrite-corrupts-bindposes|FBX binary-overwrite under a stale .meta corrupts skinned-mesh bindposes (mesh collapses to T-pose while bones animate)]] 
- [[fbx-export-standard-settings-blender-to-unity|FBX Export Standard Settings (Blender → Unity)]] [do odtworzenia]
- [[20260629-1145-blender-empties-bake-space-transform-double-axis|FBX with parent EMPTIES imports tipped 90° when exported with bake_space_transform=True]] 
- [[20260728-0915-fbx-skala-100-w-dzieciach-psuje-pomiary|FBX z Blendera: przelicznik jednostek siedzi w SKALI DZIECI, nie w korzeniu]] 
- [[forward-axis-blender-fbx-quirk|Forward Axis = -transform.right (Blender FBX Quirk)]] [szkic]
- [[freeze-inertia-tensor-not-restored|FreezeAll + automaticInertiaTensor=false Zeroes the Inertia Tensor]] [do odtworzenia]
- [[20260726-0645-image-to-3d-conditions-on-one-image|Generator obraz-do-3D karmi sie JEDNYM obrazkiem, a nie kompletem widokow]] 
- [[20260622-0951-dead-generator-vs-live-bounds|Hardcoded bounds in a (possibly dead) generator are NOT the map's real size - use the live scene]] 
- [[20260726-1420-humanoid-sloty-opcjonalne-vs-wymagane|Humanoid: sloty OPCJONALNE zwracaja null na poprawnym awatarze - fallback po nazwach nie moze byc pod jednym `!isHuman`]] 
- [[20260531-1612-quaternius-lowpoly-nature-urp-import|Importing Quaternius "Stylized Nature MegaKit" (and similar low-poly packs) into URP]] 
- [[20260728-0910-urp-jednostronne-kartki-listowia-czarne|Jednostronne kartki listowia na URP/Lit wychodza CZARNE]] 
- [[20260727-1921-kamera-z-siatki-podlogi-nie-z-sylwetki|Kamerę odtwarzaj z regularnej struktury sceny, nie z sylwetki modelu]] 
- [[20260710-1952-save-key-name-path-hash-collision|Klucz zapisu z hasha ścieżki NAZW = kolizja przy duplikatach obiektów]] 
- [[20260727-1520-loop-matching-ties-in-procedural-stitching|Least-squares loop matching is ill-conditioned when the bridge is long]] 
- [[20260622-1412-saveload-order-event-doublecount|Lekcja: licznik liczący PRZYROSTY z eventu magazynu fałszywie nalicza przy wczytaniu zapisu]] 
- [[20260702-0651-mpb-does-not-toggle-keywords|Lekcja: MaterialPropertyBlock NIE włącza keywordów shadera (emisja niewidoczna)]] 
- [[20260714-1250-ugui-okno-rosnace-pod-tresc|Lekcja: okno UI rosnace pod tresc - clamp do ekranu NIE zmniejsza tresci]] 
- [[20260721-1830-linerenderer-flat-on-surface-invisible|LineRenderer lezacy plasko na powierzchni znika, bo material jest jednostronny]] 
- [[20260715-1430-inplace-load-respawn-duplication|Load "w miejscu" bez czyszczenia = respawnowane obiekty stackuja sie z kazdym wczytaniem]] 
- [[20260628-1105-lowpoly-lake-shore-jagged-fix|Low-poly lake shore looks jagged (serrated) - submerge the rim + widen the water, don't densify]] 
- [[20260713-2130-shader-find-null-and-createprimitive-magenta-in-build|Magenta w buildzie: Shader.Find zwraca null, a CreatePrimitive daje material, którego build nie ma]] 
- [[20260727-1924-maska-sylwetki-z-dziurami-w-cieniu|Maska sylwetki może mieć dziury w środku - i przez lata tego nie widać]] 
- [[20260714-2220-maxspeed-clamp-is-not-a-speed|maxSpeed to KLAMRA, nie prędkość - pojazd i tak stanie na (napęd / tłumienie)]] 
- [[20260715-1150-meshcollider-nonreadable-native-crash|MeshCollider na siatce bez Read/Write = TWARDY natywny crash builda przy odsloniecie (nie w Edytorze)]] 
- [[20260713-1425-runtime-meshcollider-needs-readable-mesh-in-builds|MeshCollider tworzony w runtime działa w Edytorze i cicho pada w buildzie (isReadable)]] 
- [[mesh-exporter-obj-pitfalls|MeshExporter OBJ Pitfalls (3 Critical Bugs)]] [szkic]
- [[minimum-turn-factor-arcade-steering|Minimum turnFactor 0.3 for Low-Speed Arcade Steering]] [szkic]
- [[20260705-1745-mixamo-motion-only-vs-withskin-retarget|Mixamo "Without Skin" (motion-only) FBX psuje retarget Humanoid - uzyj "With Skin"]] 
- [[20260704-1045-mixamo-npc-work-carry-animations|Mixamo nie ma animacji „obsługi maszyny / pracy fizycznej" - użyj busy-idle + animacji maszyny; dla noszenia jest „Carrying"]] 
- [[20260614-1226-modal-ui-over-world-interactable-guard|Modal UI opened from a world-space interactable must guard the interaction handler]] 
- [[20260602-1730-mtree-crown-shape-crashes-native-core|MTree (modular_tree) crown_shape crashes the native C++ core - use a Ramp node into Length instead]] 
- [[20260727-1309-naprawiony-suwak-uniewaznia-strojenie|Naprawa suwaka, ktory po cichu klamal, uniewaznia CALE wczesniejsze strojenie]] 
- [[never-destructive-ops-in-play-mode|NEVER save_scene or DestroyImmediate in Play Mode]] [szkic]
- [[20260531-1705-normalize-assetpack-scale-via-modelimporter|Normalize Inconsistent Asset-Pack Scale at the Source (ModelImporter.globalScale)]] 
- [[20260624-0702-unity-new-serialized-bool-defaults-false|Nowe pole `bool` na istniejących assetach deserializuje się do `false`, nie do inicjalizatora C#]] 
- [[20260702-2200-save-system-missing-key-reset|Nowy ISaveable + stary save = przeciek żywego stanu (reset przy braku klucza)]] 
- [[20260706-1520-navmesh-raised-collider-invisible-bump|NPC chodza po "niewidzialnych gorkach": bake NavMesh z propsow + za gruby voxel nad niskopoly terenem]] 
- [[20260713-0840-symmetric-spinning-object-looks-static|Obracający się obiekt symetryczny wygląda na nieruchomy]] 
- [[20260606-1515-fbx-inplace-overwrite-fileid-preservation|Overwriting an FBX in place preserves prefab-variant references only if object NAMES are unchanged]] 
- [[20260614-2139-unity-persistence-ownership-runtime-spawned-objects|Persistence of a runtime-spawned object must be owned by its longest-living relative, never by a transient sibling]] 
- [[20260723-1746-ignorecollision-wiped-on-collider-disable|Physics.IgnoreCollision znika przy wyłączeniu collidera - dla przełączanych colliderów używaj par warstw]] 
- [[20260626-1203-fbx-pivot-direction-vs-procedural-placement|Pivot/geometry direction of an FBX must match what a procedural placement tool assumes]] 
- [[playmode-asset-pollution-vs-disk|Play-Mode in-memory edits pollute on-disk assets - and a "fix" can produce zero git diff]] [OK]
- [[20260713-0830-primitive-to-fbx-swap-kills-interaction|Podmiana prymitywu Unity na model FBX po cichu zabija interakcję]] 
- [[20260728-1320-mniej-wiekszych-kartek|Polowa kartek o rozmiarze wiekszym o jedna trzecia wyglada tak samo]] 
- [[20260726-1415-powershell-nie-czeka-na-unity-batchmode|PowerShell nie czeka na Unity.exe ani na exe gry - kontrola swiezosci builda strzela za wczesnie]] 
- [[20260722-1850-single-scene-return-to-menu-survivors|Powrot do menu glownego w grze jednoscenowej: co przezywa przeladowanie sceny]] 
- [[20260717-1430-pivot-convention-position-mismatch|Pozycja zapisana dla jednego pivota psuje sie cicho przy podmianie assetu z innym pivotem]] 
- [[20260626-1808-probe-heightfield-before-terrain-edit|Probe the real heightfield before scripting terrain edits - assumed profiles drift]] 
- [[procedural-textures-need-bake|Procedural Textures Must Be Baked Before FBX Export]] [szkic]
- [[20260615-2045-blender-voronoi-round-knots|Proceduralne okrągłe sęki w Blenderze: Voronoi F1, nie DISTANCE_TO_EDGE (+ kompensacja proporcji)]] 
- [[20260710-2140-flora-scatter-exclusion-zones|Proceduralny rozsiew dekoracji musi wykluczac pozycje obiektow interaktywnych]] 
- [[20260719-1605-paper-shell-culling-seethrough|Prześwity w modelach low-poly: najpierw sprawdź _Cull materiału, nie geometrię]] 
- [[20260610-0945-prefab-repivot-root-collider-cooked-mesh|Re-pivoting a prefab: a root MeshCollider with an embedded cooked mesh does NOT follow child-transform shifts]] 
- [[20260702-2205-ugui-rebuild-eats-clicks|Rebuild-on-event w uGUI zjada kliki + ScrollRect bez Graphica nie scrolluje]] 
- [[20260704-1732-blender-linked-basecolor-recolor|Recoloring a Blender material whose Base Color is LINKED does nothing via default_value]] 
- [[20260602-1500-mtree-nonmanifold-voxel-remesh|Reducing MTree (Modular Tree) meshes to low-poly: Decimate & Quadriflow refuse, Voxel remesh works]] 
- [[20260702-1130-blender-review-render-color-fidelity|Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"]] 
- [[20260623-0950-unity-resources-load-in-static-field-initializer-poisons-type|Resources.Load w inicjalizatorze pola statycznego psuje cały start UI (TypeInitializationException zapadkowana)]] 
- [[20260717-0010-generated-rig-bone-axis-defect-skeleton-transplant|Rigi z generatorów AI (Hunyuan): osie kości rozjechane z frontem modelu = wykrzywiona stopa w retargecie; lek = przeszczep szkieletu w Blenderze]] 
- [[20260702-1400-bark-atlas-vs-tile-strategy|Rodzina assetów współdzieląca teksturę: wybierz strategię (atlas vs kafel) PRZED budową]] 
- [[20260727-0510-rozproszenie-kartek-punkty-zaczepienia|Rozsypywanie kartek po szkielecie: licz NOSNIKI, nie kartki]] 
- [[20260713-1830-runtime-meshcollider-needs-readwrite-and-editor-cannot-prove-it|Runtime MeshCollider wymaga Read/Write Enabled - a Edytor NIE JEST w stanie tego udowodnić]] 
- [[runtime-vs-editor-script-separation|Runtime vs Editor Script Separation (Assembly Boundary)]] [szkic]
- [[20260720-1410-ortho-comparison-render-hides-occlusion|Rzut prostokątny w ujęciu porównawczym potrafi pokazać kilka obiektów nałożonych na siebie i wyglądać jak jeden poprawny obiekt]] 
- [[20260728-1105-samotest-sprawdzajacy-wlasne-normalne-jest-slepy|Samotest sprawdzajacy WLASNE normalne jest slepy na odwrocona scianke]] 
- [[20260728-1520-unity-scena-binarna-mimo-force-text|Scena zostaje BINARNA mimo Force Text, a dwa oczywiste lekarstwa nie działają]] 
- [[scene-view-ab-false-positive-game-view-ground-truth|Scene View A/B screenshots gave a false-positive diagnosis - verify in the GAME view with the live camera]] [OK]
- [[20260721-1210-screencapture-mid-frame-captures-previous-frame|ScreenCapture.CaptureScreenshotAsTexture w ciele korutyny lapie POPRZEDNIA klatke]] 
- [[20260712-1925-unity-so-asset-overrides-initializers|ScriptableObject .asset serializuje wartosci i WYGRYWA nad inicjalizatorami pol w C#]] 
- [[scriptableobject-playmode-persistence|ScriptableObject changes in Play Mode DO persist after exit]] [OK]
- [[20260605-1250-urp-flow-shader-scroll-sign|Scrolling/flow shaders: visual motion runs OPPOSITE to the flow vector]] 
- [[self-collision-compound-colliders-ignore|Self-Collision Compound BoxColliders → Physics.IgnoreCollision]] [do odtworzenia]
- [[separate-objects-mapping-rule|Separate-Objects Mapping Rule (Heightmap Limitations)]] [szkic]
- [[20260707-1315-unity-interaction-raycast-blocked-by-noninteractable-collider|Single masked Physics.Raycast for "look-at to interact" gets eaten by a non-interactable collider on a masked layer]] 
- [[20260705-2102-binary-scene-reference-check|Sprawdzanie referencji assetu w BINARNEJ scenie Unity (grep nie wystarcza)]] 
- [[stale-reflection-probe-night-whitening|Stale Skybox Reflections Whiten PBR Materials at Night (Day/Night Cycle)]] [OK]
- [[20260627-1040-daynight-editor-preview-and-pbr-water-sky-grey|Stylized PBR water looks great in editor but grey in-game - the day/night cycle drives lighting only at runtime]] 
- [[20260603-1922-model-swap-drops-scene-components|Swapping an FBX-instance GameObject silently drops its scene-side functionality]] 
- [[20260727-2320-sylwetka-nie-rozdziela-czesci-ktore-sie-stykaja|Sylwetka nie rozdziela dwóch rzeczy, które się stykają - i milczy o tym]] 
- [[20260725-1015-ai-autorig-proportions-crush-humanoid|Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke]] 
- [[tag-assignment-code-vs-inspector|Tag Assignment: Code vs Inspector for Runtime-Spawned Objects]] [szkic]
- [[20260720-1308-pula-jednoelementowa-udaje-pelne-pokrycie|Test losujacy jeden element z puli o rozmiarze 1 udaje pelne pokrycie]] 
- [[20260617-1210-tmp-text-legibility-on-textured-bg|TextMeshPro: czytelność na teksturowanym tle (drewno) + warstwy modali]] 
- [[20260730-0800-proxy-binding-unclamped-barycentric|Transfer morfa przez proxy: przycięte barycentryki RWĄ siatkę, rzut na płaszczyznę + wygładzenie działa]] 
- [[20260718-1240-pivot-vs-bounds-anchor|transform.position to PIVOT, nie geometria - kotwice wizualne licz z bounds]] 
- [[20260531-0934-tripo-polygon-soup-inverted-winding-fix|Tripo / AI-generated meshes import as "polygon soup" - see-through holes under single-sided rendering are a winding problem caused by UNWELDED verts, not interior faces]] 
- [[20260707-1320-unity-ugui-fixed-preferredheight-clips-multiline-localized-text|uGUI toast/plaque with a fixed LayoutElement.preferredHeight clips multi-line and localized text]] 
- [[20260717-1215-unity-mcp-entityid-double-precision|Unity 6.5 EntityId (64-bit) gubi precyzję w kanale MCP/JSON - generator dostaje CUDZY asset]] 
- [[20260626-1110-unity-65-material-location-migration-and-runcommand-guard|Unity 6.5: bezpieczna migracja `MaterialLocation.External` + guard w AI-Assistant Run Command]] 
- [[20260728-1545-unity65-entityid-obsolete|Unity 6.5: instance ID w SerializedProperty to teraz EntityId, a rzutowanie na int też jest zakazane]] 
- [[20260721-1040-unity-mcp-connection-revoked-is-a-lie|Unity MCP "Connection revoked" is a misleading error - the real causes are capacity limit and missing AI entitlement]] 
- [[urp-shadow-cascade-tuning|URP Shadow Cascade Tuning for Low-Poly Terrain]] [szkic]
- [[20260713-2145-urp-transparent-material-silent-failure|URP: źle skonfigurowany materiał przezroczysty to CICHA porażka, której wykrywacz magenty nie widzi]] 
- [[20260531-0934-fbx-mesh-only-verification-scan-class-names|Verifying an FBX is "mesh-only" before Mixamo: scan for the real CLASS names, not substrings - `AnimStack` matches the header property `ActiveAnimStackName`]] 
- [[vertex-color-gamma-correction-blender-to-unity|Vertex Color Gamma Correction Blender → Unity]] [do odtworzenia]
- [[20260710-1300-vertex-colors-vs-basecolor-linear|Vertex colors vs _BaseColor w Linear color space - ten sam hex renderuje się INACZEJ]] 
- [[20260612-1200-eevee-shadow-acne-wavy-lines|Wavy dark lines in EEVEE preview renders = shadow acne, not geometry]] 
- [[20260713-1030-verify-in-target-engine-not-source-tool|Weryfikuj asset w silniku DOCELOWYM, nie w narzędziu źródłowym]] 
- [[20260716-0843-generic-rig-no-retarget-foot-height|Wysokosc stop NPC: szacunek z plikow i obrys siatki KLAMIA - prawde mowia kosci stop mierzone w buildzie]] 
- [[20260707-1130-navmesh-fine-voxel-micro-gap-route-detour|Za drobny voxel NavMesh tworzy mikro-dziure, ktora ROZSPAJA trase i wymusza wielki objazd]] 
- [[20260728-1140-miernik-ktory-klamie-inaczej|Zanim zaufasz bramce, sprawdz, czy mierzy to, co widac]] 
- [[20260709-1035-stuck-static-modal-flags-unity|Zawieszone statyczne flagi modalne blokuja interakcje na stale]] 
- [[20260728-1110-meshcollider-niewypukly-z-rigidbody-gubi-kolizje|Zderzak z pelnej siatki na obiekcie z fizyka = obiekt znika ze swiata]] 
- [[20260726-1120-zero-thickness-surfaces-break-voxel-remesh|Zero-thickness surfaces make voxel remesh amputate parts of a model]] 
- [[20260719-1210-unity-build-freshness-check-dll-not-exe|Świeżość builda Unity sprawdzaj po DLL z kodem gry, nie po .exe]] 

## Silnik - wzorce (100)

Sprawdzone sposoby robienia rzeczy.

- [[four-phase-weighted-smoothstep-day-night|4-Phase Weighted Smoothstep Day/Night Transition]] [szkic]
- [[ambient-crossfade-zone-based|Ambient Crossfade Zone-Based Pattern]] [szkic]
- [[20260723-2105-twitch-anon-chat-unity|Anonimowy odczyt czatu Twitch w Unity (zero kont, kluczy i kosztow)]] 
- [[architectural-naming-convention|Architectural Elements Naming Convention]] [szkic]
- [[asset-origin-bottom-center-convention|Asset Origin at Bottom-Center Convention]] [szkic]
- [[audio-mixer-snapshots-per-game-state|Audio Mixer Snapshots per Game State]] [szkic]
- [[audio-occlusion-lpf-volume|Audio Occlusion Pattern (LPF + Volume)]] [szkic]
- [[audio-manager-mixer-architecture|AudioManager + Mixer Architecture (5 Channels + 10-Source Pool)]] [szkic]
- [[audio-reverb-zone-per-environment|AudioReverbZone per Environment]] [szkic]
- [[20260710-2250-unity-autonomous-smoke-runner-flag-file|Autonomiczny runner smoke testów w Unity: plik-flaga + plik wyników]] 
- [[awake-init-for-isaveable-with-dependencies|Awake-Init for ISaveable with Dependencies]] [szkic]
- [[migration-pattern-rollback-safety|Backend Migration Pattern with Rollback Safety]] [szkic]
- [[20260612-1340-unity-batch-fbx-import-meta-mirroring|Batch FBX import with pre-authored .meta files + prefab build in temp additive scene]] 
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] 
- [[before-delete-legacy-class-checklist|Before-Delete Legacy Class Checklist]] [szkic]
- [[20260723-1328-unity-mac-build-from-windows|Build Unity na macOS z maszyny Windows (bez Maca)]] 
- [[20260713-2150-cheat-build-extra-scripting-defines|Build z cheatami przez `extraScriptingDefines` - zamiast „odblokuj i pamiętaj cofnąć"]] 
- [[camera-lock-save-lerp-restore|Camera Lock: Save → Lerp → Restore]] [szkic]
- [[catmull-rom-spline-road-mesh|Catmull-Rom Spline + Quad Strip Mesh for Roads]] [szkic]
- [[20260611-2050-consumed-id-registry-save-pattern|Central consumed-ID registry for scene-object persistence]] 
- [[20260710-1950-central-cursor-lock-refcount|Centralny zarządca kursora z refcountem właścicieli (zamiast rozproszonych Cursor.lockState)]] 
- [[choppable-tree-multi-type-naming-convention|ChoppableTree Multi-Type Naming Convention]] [szkic]
- [[cliff-waterfall-hidden-cave|Cliff + Waterfall Hidden Cave Pattern]] [szkic]
- [[collider-distribution-rule|Collider Distribution Rule (Architecture)]] [szkic]
- [[20260609-0830-conform-terrain-to-path-via-per-x-centerline-profile|Conform a mesh terrain to a path by sampling its centreline per-axis (not a fixed scan line, not a single plane)]] 
- [[mesh-collider-convex-for-clickable-minigame-objects|Convex MeshCollider for Irregular Clickable Objects]] [do odtworzenia]
- [[crate-manager-tier-progression|CrateManager Tier Progression]] [szkic]
- [[20260606-0615-meandering-river-flow-map-baked-tangent|Curved/meandering water flow via a baked flow map (arc-length V + per-vertex tangent)]] 
- [[custom-editor-pattern-for-generators|Custom Editor Pattern for Generators]] [szkic]
- [[20260727-2145-sprawdzaj-czytnik-obrazu-renderem-wlasnego-modelu|Czytnik obrazu sprawdzaj renderem własnego modelu, nie obrazkiem, który sam sobie narysowałeś]] 
- [[dictionary-warehouse-registry|Dictionary<ProductType, int> Warehouse Registry]] [szkic]
- [[diegetic-3d-button-raycast|Diegetic 3D Button Raycast Pattern]] [szkic]
- [[discriminating-clip-vs-rig-vs-skin-humanoid-defect|Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect]] 
- [[20260625-1841-unity-collision-for-gpu-instanced-props|Dodawanie kolizji do propów rysowanych GPU instancingiem]] 
- [[fbx-long-axis-detect-programmatically|Don't assume an FBX mesh's axis - detect the longest axis programmatically from bounds]] [OK]
- [[20260625-0714-loop-fade-timefit-sfx-pattern|Dopasowanie SFX o stałej długości do akcji o zmiennej długości: pętla + wygaszenie]] 
- [[20260612-1530-fix-baked-solidify-wrong-direction|Fixing a baked-in Solidify applied in the wrong direction]] 
- [[flatten-terrain-under-road|Flatten Terrain Under Road (Smoothstep Blend)]] [szkic]
- [[footstep-raycast-surface-detection|Footstep Raycast Surface Detection]] [szkic]
- [[game-event-so-event-channel|GameEventSO ScriptableObject Event Channel]] [szkic]
- [[game-state-machine-pattern|GameStateMachine Pattern]] [szkic]
- [[get-or-add-component-pattern|GetOrAddComponent Extension Method]] [szkic]
- [[global-router-storage-pattern|GLOBAL_ROUTER Pattern (StorageManager.AddToStorage)]] [szkic]
- [[20260723-0925-unity6-display-settings-bootstrap|Globalne ustawienia ekranu w Unity 6: natywna rozdzielczosc @ cap Hz od pierwszej klatki]] 
- [[20260612-1815-headless-tmp-sdf-font-generation|Headless TMP setup: import Essentials + generate SDF font asset from editor script]] 
- [[20260622-0950-unity-map-boundary-from-bounds|Invisible map boundary from live Renderer.bounds + foot-only gap via IgnoreCollision]] 
- [[isaveable-contract|ISaveable Contract]] [szkic]
- [[20260720-1310-hash-siatki-jako-dowod-neutralnosci-refaktoru|Kanoniczny hash wyniku jako dowod neutralnosci refaktoru generatora]] 
- [[kiosk-interactable-cube-placeholder|KioskInteractable + Cube Placeholder Pattern]] [szkic]
- [[20260730-2350-layer-clearance-height-over-body|Luz między warstwami ubrań: mierz WYSOKOŚĆ NAD CIAŁEM, nie odległość do najbliższej ścianki]] 
- [[material-property-block-runtime-color-variants|MaterialPropertyBlock for Runtime Color Variants]] [szkic]
- [[20260717-1620-main-menu-single-scene-overlay|Menu glowne jako nakladka w jednej scenie (bez sceny menu)]] 
- [[mesh-collider-on-roads-stackable|MeshCollider on Roads = Stackable]] [szkic]
- [[mountains-hierarchy-front-and-backdrop|Mountains Hierarchy - Front Ring + Backdrop Double-Sided]] [szkic]
- [[20260721-1845-tint-palette-on-textured-mesh-neutralize-average|Paleta kolorow na TEKSTUROWANEJ siatce: dziel tint przez sredni kolor tekstury]] 
- [[parallel-architecture-pattern|Parallel Architecture Pattern (Locator + Events + ISaveable + Singleton)]] [szkic]
- [[20260623-1508-instanced-grass-cards|Performant stylized grass: textured cards + GPU instancing (no GameObjects)]] 
- [[20260717-1100-presave-flush-for-world-automation|Pre-save flush dla systemow automatyzacji mutujacych zapisywany swiat]] 
- [[procedural-skybox-sun-moon-trick|Procedural Skybox Sun/Moon Trick]] [szkic]
- [[20260628-1100-unity-mcp-runcommand-scene-building|Procedural Unity scene-building via unity-mcp RunCommand]] 
- [[20260625-2113-unity-procedural-boundary-align-and-gap|Procedurally generated boundary: align to a visible region + auto-gap at roads]] 
- [[20260612-0630-mountain-ring-escape-audit|Programmatic escape audit for mountain-ring map boundaries]] 
- [[20260717-0800-npc-carried-prop-upright-lock-and-axis-render|Prop niesiony przez NPC: blokada pionu w LateUpdate + render osi propa zamiast zgadywania]] 
- [[20260722-1652-npc-foot-grounding-raycast-vs-navmesh-baseoffset|Przyklejanie stop NPC do gruntu raycastem zamiast stalej korekty NavMeshAgent.baseOffset]] 
- [[quest-highlight-pattern|Quest Highlight Pattern (Quest-Flag Mechanism)]] [szkic]
- [[rack-visual-fill-alignment|Rack Visual Fill Alignment Pattern]] [szkic]
- [[20260620-1300-regenerate-so-assets-from-spec|Regeneruj assety ScriptableObject z kodowego "spec" zamiast ręcznie edytować YAML/GUID]] 
- [[20260623-0840-unity-cjk-cyrillic-fonts-tmp-and-legacy-text|Renderowanie CJK + cyrylicy w grze Unity (TextMeshPro + legacy UI.Text)]] 
- [[reputation-levels-data-driven|ReputationLevels.asset Data-Driven Progression]] [szkic]
- [[river-mesh-semi-ellipse-cross-section|River Mesh Semi-Elliptical Cross-Section]] [szkic]
- [[scriptable-object-runtime-injection|ScriptableObject Runtime Injection Pattern]] [szkic]
- [[shared-mesh-and-materials-reference|Shared Mesh + Materials Reference Pattern]] [szkic]
- [[single-material-atlas-for-static-props|Single-Material Atlas for Static Props]] [szkic]
- [[sliding-head-bandsaw-mouse-drag-tempo-minigame|Sliding Head Bandsaw - Mouse Drag Tempo Minigame]] [szkic]
- [[so-propagation-chain-via-parameters|SO Propagation Chain via Parameter Passing]] [szkic]
- [[20260713-1430-probe-visibility-by-rotating-rays-not-the-object|Sonda widoczności: obracaj PROMIENIE, nie obiekt]] 
- [[stacked-carry-system-camera-viewmodel|Stacked carry system - camera viewmodel + LIFO + species-agnostic prefab refs]] [OK]
- [[statistics-manager-pattern|StatisticsManager Pattern]] [szkic]
- [[storage-activation-gating-upgrade|Storage Activation Gating via Upgrade Purchase]] [szkic]
- [[storage-migration-primary-plus-legacy-fallback|Storage Migration: Primary New + Legacy Fallback]] [szkic]
- [[storage-rack-registry-auto-register|StorageRackRegistry Singleton + Auto-Registration via OnEnable]] [szkic]
- [[20260530-1445-mixamo-rig-swap-fix-arm-splay|Swapping a Blender-rigged humanoid for a Mixamo rig without breaking prefab/SO references]] 
- [[tool-viewmodel-child-of-camera-pattern|Tool Viewmodel as Child of Camera]] [szkic]
- [[tree-stump-state-machine-enums|TreeState + StumpState Enums State Machine]] [szkic]
- [[trunk-fall-physics-config|Trunk Fall Physics Config]] [szkic]
- [[20260724-1545-unity-photoshoot-mode-cmdline|Tryb "fotograf" w buildzie: marketingowe screenshoty bez Edytora]] 
- [[20260626-1807-bridge-abutment-seam-fix|Two-stage seam fix: terrain edge-loop + road BVH re-drape at a bridge abutment]] 
- [[typography-accessibility-stack|Typography + Accessibility Stack]] [szkic]
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] 
- [[universal-camera-lock-canmove-flag|Universal Camera Lock - canMove Flag]] [szkic]
- [[20260722-2050-unstuck-nearest-valid-ground-ring-search|Unstuck / reset: szukaj najbliższego POPRAWNEGO gruntu zamiast teleportu do bazy]] 
- [[vehicle-enter-exit-choreography|Vehicle Enter/Exit Choreography Sequence]] [szkic]
- [[vehicle-interaction-zones-as-triggers|Vehicle Interaction Zones as Triggers]] [szkic]
- [[vehicle-camera-runtime-attach-detach|VehicleCamera Third-Person Orbit (Runtime Attach/Detach)]] [szkic]
- [[vfx-performance-budget|VFX Performance Budget]] [szkic]
- [[vfx-trigger-pattern|VFX Trigger Pattern via GameEventSO]] [szkic]
- [[20260628-1405-walkable-cave-from-hollow-rock|Walkable cave from a hollow low-poly rock (don't carve the model)]] 
- [[20260728-0030-wymiar-ktorego-nie-widzi-zadne-ujecie-dopasuj-do-wszystkich|Wymiar, którego nie widzi żadne ujęcie, mierzy się dopasowaniem do wszystkich naraz]] 
- [[20260719-1605-mesh-seethrough-audit-pattern|Wzorzec audytu prześwitów w siatkach: render 3-przebiegowy > heurystyki geometryczne]] 
- [[20260702-0651-bag-pour-choreography-pattern|Wzorzec: worek wsypuje kawałki do maszyny (bag-pour choreography)]] 

## Silnik - anty-wzorce (51)

Co NIE dziala i dlaczego.

- [[20260714-2320-if-unity-editor-fixes-the-build-and-kills-the-game|`#if UNITY_EDITOR` naprawia build i po cichu zabija system]] 
- [[20260615-0913-delayed-completion-coroutine-needs-singleshot-latch|A delayed-completion coroutine that still reads input double-fires without a single-shot latch]] 
- [[20260727-1535-gates-must-not-identify-parts-by-world-coordinate|A geometry gate that identifies body parts by raw world coordinate is a gate on credit]] 
- [[20260607-2016-ugui-filled-image-needs-sprite|A UGUI `Image` with `type = Filled` but no sprite ignores `fillAmount`]] 
- [[20260713-2020-unity-binary-scene-in-git-lfs-is-irreversible-bloat|Anti-pattern: binarna scena Unity w Git LFS = nieodwracalny, rosnacy bez konca bloat]] 
- [[cycles-bake-for-solid-colors|ANTI-PATTERN: Cycles Bake for Solid Color Regions]] [szkic]
- [[20260611-1230-negative-proof-string-grep|Anti-pattern: dowodzenie "handler nie istnieje" grepem po stringu ID]] 
- [[generator-destroys-both-paths-no-guard|ANTI-PATTERN: Generator Destroys Both Paths With No Guard]] [szkic]
- [[legacy-code-conflict-after-refactor|ANTI-PATTERN: Legacy Code Conflict After Refactor]] [szkic]
- [[20260622-1345-quest-gated-pickup-softlock|Anti-pattern: pickup zużywa się bezwarunkowo, ale nadaje zdolność za bramką fazy questa]] 
- [[rotating-directional-light-day-night|ANTI-PATTERN: Rotating Directional Light for Day/Night Cycle]] [szkic]
- [[unity-runtime-writes-to-shared-material-asset|Anti-pattern: runtime writes to a shared material ASSET]] [OK]
- [[scene-files-binary-never-edit|ANTI-PATTERN: Scene Files Are Binary - Never Edit as Text]] [do odtworzenia]
- [[script-overrides-prefab-inspector-values|ANTI-PATTERN: Script Overrides Prefab Inspector Values]] [szkic]
- [[low-poly-water-side-wave|ANTI-PATTERN: sin(X) + sin(Z) Water Shader = Side Waves]] [szkic]
- [[snap-freeze-instead-of-fixing-physics-cause|ANTI-PATTERN: Snap/Freeze to Mask a Physics Bug Instead of Fixing the Cause]] [szkic]
- [[race-condition-start-vs-instantiate-parameter|ANTI-PATTERN: Start() Reads SO Before Parent Sets It]] [do odtworzenia]
- [[20260702-1135-lowpoly-thin-planar-leaves-antipattern|Anty-pattern: cienkie płaskie „soczewki" jako liście low-poly]] 
- [[20260719-1605-spawn-pool-raw-fbx-bypasses-prefab|Anty-wzorzec: pula spawnera wskazuje surowy FBX zamiast prefabu-wrappera]] 
- [[20260711-1647-blender-prop-contact-interpenetrate-not-gap|Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)]] 
- [[20260714-1245-test-bez-trybu-porazki|Anty-wzorzec: test, ktory nie ma jak zawiesc (silnik "naprawia" mierzona wielkosc)]] 
- [[20260723-2140-silent-loc-fallback-antipattern|Cichy fallback lokalizacji ukrywa nieprzetłumaczoną treść]] 
- [[20260710-2115-collider-from-first-meshfilter-antipattern|Collider z GetComponentInChildren&lt;MeshFilter&gt; na wielosiatkowym FBX = collider z fragmentu modelu]] 
- [[20260606-0930-baked-atlas-texture-foreign-uvs|Don't apply a baked-atlas texture to a mesh whose UVs were authored for a different atlas]] 
- [[20260613-0610-dim-scrim-must-not-reuse-9slice-panel-factory|Don't build a full-screen dim/scrim by reusing your skinnable panel factory]] 
- [[20260716-0843-dual-owner-persistence-duplication|Dwoch wlascicieli trwalosci jednego obiektu w save (rejestr + rekonstrukcja stanu)]] 
- [[20260624-0813-unity-setup-scripts-rebuild-reposition-antipattern|Edytorowe skrypty „Setup X" które DestroyImmediate + Instantiate + ustawiają pozycję = niszczące - nie używaj ich do drobnych poprawek]] 
- [[20260713-1200-silent-null-guard-hides-dead-ui|Fabryka, która po cichu zwraca obiekt bez wymaganego dziecka]] 
- [[20260718-1030-particlesystemrenderer-bounds-trap|GetComponentsInChildren<Renderer> + Encapsulate(bounds) psuje sie po dodaniu czasteczek]] 
- [[20260716-1814-inplace-load-orphaned-satellite-objects|In-place load osieroca obiekty-satelity (dziura po wczytaniu stała na dorosłym drzewie)]] 
- [[20260727-0525-jeden-suwak-dwie-role|Jeden suwak sterujacy dwiema roznymi rzeczami]] 
- [[20260712-1820-save-migration-schema-version-gate|Jednorazowa migracja zapisu MUSI być bramkowana wersją schematu, nie obecnością/brakiem migrowanego wpisu]] 
- [[20260713-1845-monobehaviour-class-must-match-filename|Klasa MonoBehaviour dołożona przez AddComponent MUSI leżeć w pliku o swojej nazwie - inaczej scena psuje build]] 
- [[20260702-1957-lazy-singleton-ondestroy-teardown|Lazy-create singleton wołany z OnDestroy tworzy obiekty przy zamykaniu sceny]] 
- [[20260720-0915-loft-nie-da-plaskiej-szyby|Malowanie szyby kolorem na powierzchni z loftu nigdy nie da płaskiej tafli]] 
- [[20260608-1503-mcp-scene-capture-omits-gizmos|MCP scene-capture tools render geometry only - they do NOT show editor gizmos / Handles.Label]] 
- [[20260730-1710-blender-materials-clear-resets-face-indices|Mesh.materials.clear() zeruje material_index na ściankach]] 
- [[20260730-1217-bone-side-names-vs-axis-sign|Nie zgaduj strony ciała ze znaku osi ani z nazwy kości (.L/.R)]] 
- [[20260703-0900-slot-unlocks-positional-not-id|Numer w id odblokowania ≠ pozycja slotu (sloty kupowane w dowolnej kolejności)]] 
- [[20260614-1343-singleton-oneshot-flag-bleed|One-shot input flag on a persistent singleton bleeds across re-entries]] 
- [[20260702-1610-fake-null-so-null-conditional-trap|Operator `?.` NIE chroni referencji Unity - brakujący asset SO wybucha NRE w środku kanału eventowego]] 
- [[20260730-2140-shape-key-layer-corrections-oscillate|Poprawki anty-przebiciowe w danych POJEDYNCZEGO klucza kształtu oscylują]] 
- [[20260724-1817-diegetic-button-overlap-steal|Powiekszone collidery klikania ciasnych guzikow 3D + wybor "pierwsze trafione pudelko" = sasiad kradnie klik]] 
- [[20260612-1330-getbuiltinresource-extra-null|Resources.GetBuiltinResource zwraca NULL dla sprite'ów UI (builtin-EXTRA) - pasek Filled rysuje się jako pełny quad]] 
- [[20260721-0715-sentinel-value-as-mode-flag|Sentinel value: wnioskowanie TRYBU z mierzonej liczby zamiast jawnej flagi]] 
- [[20260723-1747-spawn-y-tunneling-oneside-terrain|Spawn na Y rodzica + jednostronny teren zerowej grubości + Discrete = obiekty pod mapą]] 
- [[20260721-1215-ui-fit-check-measuring-rect-instead-of-text|Sprawdzanie "czy tekst sie miesci" przez pomiar RectTransform]] 
- [[20260723-1215-ugui-mask-clear-image-invisible|uGUI: Mask z obrazkiem Color.clear = NIEWIDZIALNA zawartosc (a transformy klamia, ze wszystko gra)]] 
- [[20260710-2010-material-props-wrong-shader-inert|Ustawianie właściwości materiału bez sprawdzenia SHADERA = ciche nic + ryzyko nadpisania]] 
- [[20260720-1306-walidator-spelniony-przez-konstrukcje|Walidator, ktory jest spelniony automatycznie przez konstrukcje]] 
- [[20260719-1322-minigame-abort-latch-timing|Zatrzask "punkt bez odwrotu" ustawiany przy wejsciu w faze zamiast przy akcji gracza]] 

## Tycoon - wzorce (27)

Wzorce specyficzne dla gatunku.

- [[20260704-2030-tycoon-economy-two-clock-balancing|Balansowanie ekonomii progresu metodą „dwóch zegarów" + koperty przychodu]] 
- [[carry-capacity-progression-sprint|Carry Capacity Progression + Sprint Advantage]] [szkic]
- [[20260629-1917-diegetic-buttons-frame-minigame|Console buttons FRAME a skill minigame (they're flow-control, not the mechanic)]] 
- [[customer-tier-system|Customer Tier System (Regular / Contractor / VIP)]] [szkic]
- [[debris-cleanup-single-click-drop|Debris Cleanup - Single-Click Drop Materials Visual]] [szkic]
- [[20260607-1233-warehouse-filtered-order-pool|Gate a content pool by runtime availability, not explicit unlock flags]] 
- [[initial-fill-on-load|Initial Fill on Load (Don't Serialize NPC State)]] [szkic]
- [[20260716-0843-value-greedy-basket-priciest-dominates|Koszyk dobijany do kwoty "krokiem najblizej celu" = najdrozszy produkt dominuje kazde zamowienie]] 
- [[20260710-1030-supply-weighted-orders-need-floor|Losowanie zamówień ważone podażą wymaga PODŁOGI wag]] 
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
- [[20260702-0651-batch-machine-loop-pattern|Wzorzec: pętla serii maszyny z minigrą (batch-machine loop)]] 

## Tycoon - decyzje (12)

Decyzje projektowe Kerf.

- [[building-progression-instant-spawn|Building Progression - Instant Spawn Post-Purchase]] [szkic]
- [[loading-station-decision|LoadingStation Decision - Manual Walk Early, Station Late]] [szkic]
- [[planting-spot-universal-not-typed|PlantingSpot Universal (Not Typed by Species)]] [szkic]
- [[player-built-vs-purchased-dichotomy|Player-Built vs. Purchased Dichotomy]] [szkic]
- [[quantity-not-quality-principle|Quantity-Not-Quality Design Principle]] [szkic]
- [[rack-architecture-decision|Rack Architecture Decision (3 Options)]] [szkic]
- [[sales-flow-decision-hybrid|Sales Flow Decision - Hybrid D (Player + NPC Side by Side)]] [szkic]
- [[tier-system-foundation|Tier System Foundation - Trees, Products, Machines (SOURCE OF TRUTH)]] [szkic]
- [[tool-tier-replacement-not-inventory|Tool Tier Replacement - NOT Multiple Tiers in Inventory]] [szkic]
- [[ui-phase-4-catalog|UI Phase 4 Catalog (12 Systems)]] [szkic]
- [[vfx-wycofane-decision|VFX Wycofane - Sawdust, Kurz, Liście Cut from MVP]] [szkic]
- [[zero-code-changes-philosophy|Zero-Code-Changes Philosophy]] [szkic]

## Tycoon - lekcje (2)

- [[20260706-1215-supply-estimator-must-honor-unlock-gate|Estymator podaży/dostepnosci MUSI respektowac te sama bramke odblokowan co produkcja]] 
- [[20260722-1722-score-exactly-what-the-hud-promises|Punktuj DOKLADNIE te wielkosc, ktora pokazujesz graczowi]] 

## Tycoon - anty-wzorce (3)

- [[20260714-2210-minigame-abort-refund-free-reroll|Minigra zużywa surowiec na STARCIE, a przerwanie go zwraca]] 
- [[20260712-0010-per-sku-price-decay-rotation-exploit|Per-SKU price decay przegrywa z rotacja 2 SKU (gdy odnowa wyprzedza tempo produkcji)]] 
- [[20260714-2215-order-value-topdown-makes-prices-meaningless|Zamówienie losowane OD KWOTY w dół - cennik przestaje cokolwiek znaczyć]] 

## Survival - wzorce (1)

Pod przyszly projekt.

- [[chunk-based-world-loading|Chunk-Based World Loading (Eskimo Simulator)]] [szkic]

## Survival - decyzje (1)

- [[multiplayer-from-mvp-not-retrofit|Multiplayer from MVP - Not Retrofit]] [szkic]

## Miedzy gatunkami - decyzje (1)

- [[audio-strategy-minimal-music-heavy-ambient|Audio Strategy - Minimal Music + Heavy Ambient + Voice Bites]] [szkic]

## Workflow - jakosc i weryfikacja (13)

Zasady, na ktorych stoi caly system bramek.

- [[20260717-1115-style-match-real-assets-not-description|Anti-pattern: dopasowywanie stylu do OPISU stylu zamiast do prawdziwych assetów z gry]] 
- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] [OK]
- [[20260720-0920-red-proof-musi-uzbroic-wszystkie-checki|Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony]] 
- [[build-is-the-only-truth-editor-lies|Edytora nie da sie oszukac, zeby udawal build]] [OK]
- [[20260716-1816-runtime-redproof-flag-for-build-probes|Flaga runtime "-redproof" zamiast drugiego builda do czerwonego dowodu sondy]] 
- [[20260726-2030-mierz-wzorzec-zamiast-zgadywac-proporcje|Gdy rezyser mowi "wyglada zle", zmierz cos, co juz zaakceptowal]] 
- [[20260728-1900-miara-z-degeneracja|Miara optymalizowana samotnie znajduje rozwiazanie zdegenerowane]] 
- [[20260720-1415-calibrate-new-validator-on-approved-artifact|Nowy walidator uruchom najpierw na ZATWIERDZONYM artefakcie; jeśli tam zapala się na czerwono, błędny jest walidator, nie artefakt]] 
- [[20260725-1845-silhouette-measurement-catches-shadow-and-vignette|Pomiar proporcji z renderu klamie trzy razy, zanim zacznie mowic prawde]] 
- [[20260728-1500-bramka-ponad-sufitem|Prog bramki ponad zmierzonym sufitem zamienia kazda runde w porazke]] 
- [[20260713-1900-build-early-never-built-project-hides-editor-only-bugs|Projekt, który nigdy nie był budowany, hoduje całą klasę uśpionych błędów]] 
- [[20260722-1652-relative-only-test-blind-to-common-mode-error|Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)]] 
- [[20260714-2245-unity-batchmode-returns-before-build-finishes|Unity w trybie wsadowym WRACA, zanim build się skończy - i sonda daje fałszywe zielone światło]] 

## Workflow - Claude Code (15)

Agenci, hooki, skille, protokoly.

- [[do-not-move-hardcoded-positions|"DO NOT MOVE" Hardcoded Positions Convention]] [szkic]
- [[clear-vs-compact-decision-rules|/clear vs /compact - Decision Rules]] [szkic]
- [[backup-scene-before-modify|Backup Scene Before Structural Modification]] [szkic]
- [[console-collapse-loop-suffix|Console Collapse + Loop Suffix Convention]] [szkic]
- [[context-degradation-threshold|Context Degradation Threshold (20-40% Remaining)]] [szkic]
- [[cross-project-stack-reuse|Cross-Project Stack Consistency (TT → Eskimo)]] [szkic]
- [[intentionally-low-maxcapacity-test-racks|Intentionally Low maxCapacity for Test Racks]] [szkic]
- [[iterative-checkpoint-workflow|Iterative Checkpoint Workflow for Generated Assets]] [szkic]
- [[20260614-1940-claude-code-local-plugin-persistence|Loading a LOCAL Claude Code plugin permanently (marketplace, not --plugin-dir)]] 
- [[quad-backtick-prompt-format|Quad-Backtick Claude Code Prompt Format]] [szkic]
- [[read-actual-code-before-hypothesizing|Read Actual Code Before Hypothesizing]] [szkic]
- [[scene-attachment-check-before-deleting|Scene Attachment Check Before Deleting MonoBehaviour]] [szkic]
- [[stop-hook-infinite-loop-risk|Stop Hook Infinite Loop Risk]] [do odtworzenia]
- [[three-level-analysis-system|Three-Level Analysis System]] [szkic]
- [[universal-cleanup-post-migration|Universal Cleanup Post-Migration]] [szkic]

## Workflow - narzedzia MCP (12)

Coplay, Blender, ElevenLabs.

- [[20260610-1820-blender-mcp-failure-headless-fallback|blender-mcp bridge failure modes + headless CLI fallback]] 
- [[blender-mcp-interactive-remodel-loop|Blender-MCP Interactive Remodel Loop (GUID-Preserving In-Place Replace)]] 
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] 
- [[20260611-coplay-set-property-color-json-silent-white|Coplay set_property: Color fields need comma-separated r,g,b,a - JSON silently writes white]] 
- [[20260725-0620-mcp-port-9876-protocol-collision|Dwa serwery MCP na porcie 9876: "Incomplete JSON response" to konflikt protokolu, nie zawieszony Blender]] 
- [[20260531-1500-mixamo-clean-mesh-extraction|Extract a clean mesh-only FBX from a rigged source for Mixamo re-rig]] 
- [[20260606-1628-mcp-scene-capture-renders-main-scene-not-prefab-stage|MCP Scene-Capture Renders the Active Scene, Not an Open Prefab Stage]] 
- [[mcp-wildcard-permissions-format|MCP Wildcard Permissions Format]] [do odtworzenia]
- [[skill-loading-on-demand-vs-reference|Skill Loading: @Reference vs On-Demand]] [do odtworzenia]
- [[20260702-1612-editor-probes-return-result-not-logs|Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log]] 
- [[20260728-0900-unity-cli-pipeline-as-agent-bridge|Unity CLI + com.unity.pipeline jako most agenta do żywego edytora]] 
- [[20260728-0905-closed-plugin-changelog-by-dll-string-diff|Wtyczka bez changelogu: różnica ciągów znaków w DLL zamiast zgadywania]] 

## Workflow - pipeline assetow (11)

Tripo, Blender, Unity.

- [[audio-asset-pipeline|Audio Asset Pipeline (ElevenLabs + Suno + FFmpeg)]] [szkic]
- [[character-pipeline-tripo-mixamo-unity|Character pipeline: Tripo mesh → Mixamo rig → Unity (clean, working recipe)]] 
- [[flatten-must-be-baked-into-geometry-when-code-forces-uniform-scale|Flatten Must Be Baked Into Geometry When Code Forces Uniform Scale]] 
- [[20260606-1632-in-place-fbx-overwrite-static-vs-rigged|In-Place FBX Overwrite: Safe for Static Meshes, Dangerous for Rigged]] 
- [[20260719-2015-ai-gen-model-geometry-debt|Modele generowane przez AI (Tripo/Hunyuan): dług geometryczny - czasem taniej zbudować od zera]] 
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] 
- [[polybrush-iteration-rule|Polybrush Iteration Rule - No Return to Generator]] [szkic]
- [[polybrush-settings-low-poly|Polybrush Settings for Low-Poly Terrain]] [szkic]
- [[20260722-1145-loop-seam-measure-and-crossfade|Pętla dźwiękowa: zmierz styk, przenikaj ogon w początek, dopisz próg do sondy]] 
- [[tripo-asymmetric-floating-retopo|Tripo Asymmetric / Floating Elements - Blender Retopo]] [szkic]
- [[tripo-cleanup-pipeline|Tripo Cleanup Pipeline]] [szkic]

## Workflow - modele 3D (11)

Modelowanie, UV, wypalanie.

- [[blender-headless-python-generation|Blender Headless Python Script Generation]] [szkic]
- [[20260612-1845-blender-9slice-ui-sprites|Blender-rendered 9-slice-ready UI sprites (3D panel → ortho render → Unity Sliced sprite)]] 
- [[20260727-1545-measure-the-reference-dont-guess-proportions|Measure the reference from pixels, then gate against the measurement]] 
- [[20260726-1535-blender-addon-parametryczny-suwaki|Parametryczny dodatek do Blendera: trzy pulapki, ktore kosztuja godzine kazda]] 
- [[20260725-1830-plaskie-tekstury-z-plam-referencji|Plaskie tekstury dla modelu z generatora 3D: tozsamosc elementu bierz z REFERENCJI, nie z koloru]] 
- [[procedural-textures-cycles-commercial|Procedural Textures in Blender Cycles (Commercial Release Rationale)]] [szkic]
- [[terrain-skirt-against-seethrough-gap|Terrain Skirt Against the See-Through Gap]] 
- [[tripo-organic-vs-blender-geometric-decision|Tripo (organic) vs Blender MCP (geometric) - Pipeline Routing Decision]] [szkic]
- [[tripo-vocab-firewood-wedge|Tripo Vocab: Firewood = Wedge-Shaped Piece]] [szkic]
- [[20260725-2050-kontrakt-liczbowy-bez-nazw-osi|Wspolny kontrakt liczbowy dla kilku agentow, ktory nie nazywa osi]] 
- [[zero-floating-zero-flickering-mandate|ZERO Floating / ZERO Flickering Mandate]] [szkic]

## Workflow - pozostale (15)

Narzedzia, proces, integracje, wydania.

- [[20260727-1422-bramka-musi-umiec-zaliczyc-nie-tylko-oblac|Bramka musi mieć udowodniony tryb ZALICZENIA, nie tylko PORAŻKI]] 
- [[20260723-1325-ps51-compress-archive-mac-zip|Compress-Archive (PS 5.1) do paczek dla macOS]] 
- [[20260613-1030-dont-lfs-commit-unity-timestamped-backups|Don't commit timestamped Unity scene/asset backups into git-LFS]] 
- [[20260718-0805-headless-visual-proof-batchmode|Dowod wizualny z Unity batchmode (bez otwierania Edytora)]] 
- [[20260725-1830-image-model-cannot-force-figure-proportions|Generator obrazkow nie da sie zmusic opisem do proporcji figury (7,5 glowy)]] 
- [[20260713-2025-ff-merge-without-checkout-protects-dirty-worktree|Pattern: przewiniecie galezi BEZ checkoutu (`git fetch . src:dst`)]] 
- [[20260727-0900-rekonstrukcja-z-referencji-granica-prawna|Referencja a kopia: gdzie przebiega granica przy odtwarzaniu cudzego modelu]] 
- [[20260531-1614-editor-flora-scatter-patterns|Reproducible Editor Flora Scatter onto a Mesh Terrain]] 
- [[20260722-1610-powershell51-ansi-mangles-utf8-assets|Skrypt PowerShell 5.1 psuje polskie znaki przy masowej edycji plików assetów]] 
- [[20260723-1940-telemetry-paired-feedback-triage|Triage feedbacku testera w parze z telemetria i logiem]] 
- [[20260724-1120-twitch-wss-one-frame-one-command|Twitch IRC po WebSocket: jedna paczka = jedna komenda (i jak cichy klient to ukryl)]] 
- [[20260624-0702-powershell5-ps1-ansi-unicode-data-file|Windows PowerShell 5.1 czyta `.ps1` jako ANSI - literały Unicode w skrypcie się sypią]] 
- [[20260724-1907-steam-release-timeline-two-reviews-hard-clock|Wydanie gry na Steam: sekwencyjne recenzje + twardy zegar Coming Soon]] 
- [[20260726-1930-zielone-bramki-nie-dowodza-ze-wyglada-dobrze|Zielona tablica bramek nie dowodzi, ze cos wyglada dobrze]] 
- [[20260722-1950-project-wide-rename-safely|Zmiana nazwy w calym projekcie bez psucia zapisow i kodowania plikow]] 

## Projekty (1)

Indeksy projektowe.

- [[timber-tycoon|Timber Tycoon - Project Index]] [OK]

## Pozostale (11)

- [[CLAUDE-md-gra-przed-zmiana-poziomow-2026-07-30|CLAUDE-md-gra-przed-zmiana-poziomow-2026-07-30]] [szkic] (`_archive`)
- [[KB_BUILD_PACKAGE|KB Build Package — Timber Tycoon Knowledge Base]] [szkic] (`_archive`)
- [[MOC-stary-linki-github-2026-07-30|Map of Content — Knowledge Base]] [szkic] (`_archive`)
- [[20260530-1600-mixamo-foot-asymmetry-clip-vs-rig-discriminator|Discriminating CLIP vs RIG vs SKIN for a one-sided humanoid animation defect]] [zastapione] (`_archive/duplicates`)
- [[20260530-1800-fbx-binary-overwrite-corrupts-skin-bindposes|Never binary-overwrite a skinned FBX under an existing .meta — it desyncs bindposes]] [zastapione] (`_archive/duplicates`)
- [[20260531-0700-fbx-binary-overwrite-corrupts-bindposes|Binary-overwriting an FBX under its existing .meta corrupts skinned-mesh bindposes (T-pose collapse)]] [zastapione] (`_archive/duplicates`)
- [[20260610-0950-coplay-roslyn-diagnostic-crash-workaround|Coplay execute_script crashes on ANY compile diagnostic — compile via Unity + reflection trigger instead]] [zastapione] (`_archive/duplicates`)
- [[anti-pattern|<What NOT to do>]] [szkic] (`templates`)
- [[decision|<Decision title>]] [szkic] (`templates`)
- [[lesson|<Title - specific, searchable>]] [szkic] (`templates`)
- [[pattern|<Pattern name>]] [szkic] (`templates`)

## Skrzynka wejsciowa (2)

Drafty czekajace na weryfikacje. Nie sa indeksowane tutaj pojedynczo -
kolejka do przegladu jest w [[PRZEGLAD-INBOXU]].

---

Zaindeksowano 445 wpisow uporzadkowanych. Mapa jest generowana skryptem
`tools/kb_audit.py --write-moc` - nie edytuj jej recznie, bo zmiany przepadna.
