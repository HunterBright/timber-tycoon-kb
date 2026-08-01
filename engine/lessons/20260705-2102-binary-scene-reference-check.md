---
title: Sprawdzanie referencji assetu w BINARNEJ scenie Unity (grep nie wystarcza)
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-05'
project: Kerf - Sawmill Tycoon
tags:
- unity
- scene
- binary-serialization
- asset-deletion
- references
- mcp
- editor-script
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Sprawdzanie referencji assetu w BINARNEJ scenie Unity (grep nie wystarcza)

## Kontekst / problem
Przed usunieciem osieroconego assetu standardowo grepuje sie GUID z .meta po plikach .unity/.prefab/.asset (tekst YAML). Gdy scena jest zapisana BINARNIE (Force Text wylaczony, albo duza scena), grep GUID zwraca 0 trafien FALSZYWIE - binarny format nie trzyma GUID jako ASCII hex, wiec grep go nie widzi. Ryzyko: usuniecie assetu realnie uzywanego przez obiekt w scenie -> missing reference.

Sygnal ze scena jest binarna: `head` pliku .unity zaczyna sie od bajtow niedrukowalnych, nie od `%YAML 1.1`.

## Rozwiazanie (zwalidowane)
Referencje w otwartej scenie sprawdz editor-scriptem (przez MCP/Coplay execute_script), nie grepem:
1. Zaladuj asset(y): `AssetDatabase.LoadAllAssetsAtPath` (lapie tez sub-assety, np. meshe FBX) + `LoadMainAssetAtPath`.
2. Iteruj WSZYSTKIE komponenty sceny: `FindObjectsByType<Component>(FindObjectsInactive.Include, ...)`.
3. Dla kazdego `new SerializedObject(comp)` + `GetIterator()` -> gdy `propertyType == ObjectReference` i `objectReferenceValue` pasuje do targetu -> to referencja. Lapie materialy na rendererach, meshe na MeshFilter, itd.
4. Zlicz trafienia; 0 = bezpieczne usuniecie.

Analogicznie: przynaleznosc komponentow do sceny sprawdzaj `GameObjectUtility.GetMonoBehavioursWithMissingScriptCount` + `RemoveMonoBehavioursWithMissingScript`.

## Lekcja
Delete-safety w Unity musi obejmowac scene binarna osobnym torem (editor-script), bo tekstowy grep jest slepy na binarny .unity. Taniej zweryfikowac 5 min skryptem niz debugowac missing-ref po fakcie.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260624-0813-unity-setup-scripts-rebuild-reposition-antipattern|Edytorowe skrypty „Setup X" które DestroyImmediate + Instantiate + ustawiają pozycję = niszczące - nie używaj ich do drobnych poprawek]] - wspolne: scene, editor-script
- [[20260531-1610-coplay-execute-script-masks-compile-errors|Coplay `execute_script` Hides Compile Errors - Use Unity-Compiled Editor Scripts Instead]] - wspolne: editor-script, mcp
<!-- /POWIAZANE:auto -->
