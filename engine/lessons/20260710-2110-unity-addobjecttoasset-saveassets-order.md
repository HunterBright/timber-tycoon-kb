---
title: AddObjectToAsset wymaga SaveAssets() PRZED ImportAsset(), inaczej sub-asset znika
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-10'
project: Timber_Tycoon
tags:
- unity
- editor-scripting
- assetdatabase
- prefab
- sub-asset
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# AddObjectToAsset wymaga SaveAssets() PRZED ImportAsset(), inaczej sub-asset znika

## Kontekst
Skrypt edytorowy osadzal wygenerowana siatke kolizyjna (Mesh) w pliku .prefab jako sub-asset
(konwencja: collider mesh trzymany lokalnie w prefabie, nie w FBX).

## Problem
Sekwencja `AssetDatabase.AddObjectToAsset(mesh, path)` -> `AssetDatabase.ImportAsset(path)`
gubi obiekt: AddObjectToAsset dziala tylko w pamieci, a ImportAsset czyta plik Z DYSKU -
czyli wersje sprzed dodania. Po imporcie sub-assetu nie ma (LoadAllAssetsAtPath go nie widzi),
bez zadnego bledu w konsoli. Objaw: 0/6 prefabow naprawionych, "sub-asset nie przetrwal importu".

## Rozwiazanie
Kolejnosc: `AddObjectToAsset(mesh, path)` -> `AssetDatabase.SaveAssets()` (zapis na dysk)
-> `ImportAsset(path)` (stabilizacja fileID) -> ponownie pobrac referencje przez
`LoadAllAssetsAtPath` (stara referencja moze byc nieaktualna po imporcie) -> dopiero teraz
przypisac do komponentu i zapisac prefab (LoadPrefabContents/SaveAsPrefabAsset).

## Reguła
Kazdy AddObjectToAsset musi byc odprowadzony przez SaveAssets() zanim cokolwiek wymusi
re-import lub odczyt pliku; po imporcie nie ufac starym referencjom C# do sub-assetu.
