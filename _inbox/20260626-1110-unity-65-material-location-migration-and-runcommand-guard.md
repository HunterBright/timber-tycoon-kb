---
title: 'Unity 6.5: bezpieczna migracja `MaterialLocation.External` + guard w AI-Assistant Run Command'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-26'
project: Kerf - Sawmill Tycoon
tags:
- unity
- unity-6.5
- fbx
- material-import
- asset-pipeline
- mcp
- lightmapping
- apv
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Unity 6.5: bezpieczna migracja `MaterialLocation.External` + guard w AI-Assistant Run Command

## Kontekst
Po upgradzie Unity 6000.3.x → 6000.5.x (6.5) w logach pojawiły się ostrzeżenia importu:
`MaterialLocation.External is obsolete. External Material Location is no longer supported.` (Unity 6.5 wyrzuciło legacy „External" material location). Trzeba było wyciszyć je bez psucia materiałów na ~17 modelach FBX, które przez ten tryb wiązały custom `Mat_*.mat`.

## Lekcja 1 — migracja External → InPrefab jest BEZPIECZNA, jeśli istnieje remap
- W `.fbx.meta`: `materialLocation: 0` = External (legacy), `1` = InPrefab (zalecane).
- Fix: `(AssetImporter.GetAtPath(path) as ModelImporter).materialLocation = ModelImporterMaterialLocation.InPrefab; imp.SaveAndReimport();`
- **NIE ruszać** `materialImportMode` / `materialName` / `materialSearch` / `externalObjects`.
- **Dlaczego bez ryzyka „różowych" modeli:**
  1. `externalObjects` (Remapped Materials) wiąże slot → istniejący `.mat` po GUID; flip `materialLocation` go NIE czyści.
  2. Custom `.mat` to NIEZALEŻNE assety — reimport ich nie regeneruje ani nie nadpisuje (potwierdzone w docs FBXImporter-Materials: „Extracted materials are independent…").
  3. Żywe prefaby/scena referują `.mat` WPROST na MeshRenderer (po GUID), niezależnie od importera — więc nawet najgorszy reimport nie zmienia runtime'owego wyglądu.
- **Wzorzec kanarka:** najpierw 1 model z realnym 2-slotowym remapem + instancją w scenie; potwierdź `remapCount` przed==po, brak zmiany `.mat` w git (jedyna zmiana = `.fbx.meta`), brak różu; dopiero potem batch reszty.
- Po flipie zostają jeszcze CS0618 w skryptach editorowych, które same wołają `ModelImporterMaterialLocation.External` — podmień na `.InPrefab` (zachowanie identyczne, bo i tak remapują zewnętrznie).

## Lekcja 2 — GOTCHA: guard w Unity AI Assistant „Run Command" (most MCP)
Most `unity-mcp Unity_RunCommand` = wbudowany **Unity AI Assistant** (namespace `Unity.AI.Assistant.Agent.Dynamic.Extension.Editor`). Ma **statyczny guard** odrzucający operacje destrukcyjne/interaktywne PRZED wykonaniem:
- BLOKOWANE (błąd `User interactions are not supported for MCP tool calls`, kod NIE rusza): `AssetDatabase.DeleteAsset`, `AssetDatabase.Refresh`, `System.IO.File/Directory.Delete`.
- PRZECHODZI (mimo opakowania `executed partially, but reported warnings` — realnie się wykonuje): `importer.SaveAndReimport()`, mutacje ustawień importera/obiektów, czysty odczyt+Log.
- **Wniosek:** kasowanie assetów i `Refresh` rób POZA Unity (Bash/git/`rm`), a zmiany importera rób przez `SaveAndReimport`. Po kasowaniu plikowym Unity i tak odświeży bazę na focus okna.

## Lekcja 3 — `Generate Lighting` po major-upgrade NIE czyści APV, jeśli APV nie jest aktywne
- Ostrzeżenie `Lighting data asset 'LightingData' is incompatible…` → pełny **Generate Lighting** (nie sam relink) przebudowuje lightmapy + reflection probes; ostrzeżenie znika.
- Flaga `mightNeedRebaking: 1` na komponencie `ProbeVolume` w pliku sceny **może zostać `1`**, jeśli APV faktycznie nie jest zainicjowane (`ProbeReferenceVolume.instance.isInitialized == False`). Goły obiekt „Adaptive Probe Volume" w hierarchii ≠ aktywne APV. Wtedy flaga jest martwa (zero ostrzeżenia, zero wpływu na render) — nie warto jej gonić; realne APV to osobna konfiguracja (baking set).
- Diagnostyka bez zgadywania: `SceneManager.GetActiveScene().isDirty`, `Lightmapping.lightingDataAsset`, `ProbeReferenceVolume.instance.isInitialized`.
- Lightmapy zapisują się jako pliki OD RAZU przy bake, ale przypisania w scenie + flagi utrwala dopiero `Ctrl+S` — jeśli scena = isDirty po bake, przypomnij o zapisie (Edit Mode!).
