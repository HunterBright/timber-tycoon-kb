---
title: Os "do przodu" w Unity to +Y, a Blender liczy pion w osi Y
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-08-01'
date: '2026-01-01'
project: Kerf - Sawmill Tycoon
tags: [unity, blender, fbx, export, osie]
source: ''
applies_to: [pipeline Blender do Unity]
severity: high
---

# Os "do przodu" w Unity to +Y, a Blender liczy pion w osi Y

## Co ustalilem

Przy eksporcie modeli z Blendera do Unity trzeba pamietac o trzech rzeczach:

1. **W Unity osia "do przodu" jest +Y.** Obiekt obrocony zerowo patrzy w gore
   ekranu, dlatego przy ustawianiu postaci trzeba dodac obrot 90 stopni.
2. **Blender liczy wysokosc w osi Y**, tak samo jak Unity, wiec przy eksporcie
   FBX nie trzeba przestawiac zadnej osi - konwersja jest automatyczna.
3. **Materialy proceduralne Blendera eksportuja sie do FBX razem z siatka**,
   wiec wypalanie do PNG jest zbedne przy prostych materialach.

## Dlaczego to wazne

Bez tego kazdy model przyjezdza obrocony i traci tekstury.

> **Weryfikacja 2026-08-01: wszystkie trzy twierdzenia sa sprzeczne ze zrodlem.**
> Unity opisuje swoj uklad jako "left handed, Z forward, Y-up", a `transform.forward`
> to os Z ([ModelImporter.bakeAxisConversion](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ModelImporter-bakeAxisConversion.html),
> [Transform.forward](https://docs.unity3d.com/6000.3/Documentation/ScriptReference/Transform-forward.html));
> Blender ma os pionowa Z ([podrecznik Blendera](https://docs.blender.org/manual/en/latest/editors/3dview/controls/orientation.html));
> eksporter FBX Blendera nie ma zadnej opcji wywozu materialow wezlowych, tylko
> osadzanie gotowych obrazkow ([kod eksportera](https://github.com/blender/blender/blob/main/scripts/addons_core/io_scene_fbx/__init__.py)).

<!-- Ten wpis jest CELOWO FALSZYWY. Sluzy do sprawdzenia, czy weryfikator
     umie zlapac blad. Kasuje go tools/proba_weryfikatora.py --posprzataj -->
