---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [unity, marketing, screenshots, automation, steam, command-line, build]
date: 2026-07-24
status: draft
---

# Tryb "fotograf" w buildzie: marketingowe screenshoty bez Edytora

## Problem
Steam wymaga screenshotow 1920x1080+ i grafik-okladek. Zrzuty z Edytora odpadaja
(gizmosy, watermarki, inna jakosc), a reczne pozycjonowanie kamery w grze jest
niepowtarzalne (po kazdej poprawce sceny trzeba celowac od nowa).

## Wzorzec
MonoBehaviour bootstrapowany argumentem wiersza polecen (wzor: sonda build-smoke):
1. `[RuntimeInitializeOnLoadMethod(AfterSceneLoad)]` + bramka `HasArg("-photoshoot")`;
   arg dopisany do centralnej bramki trybow testowych (bypass menu glownego).
2. Lista ujec w JSON OBOK EXE (pozycja, euler, fov, godzina, supersize) - iteracja
   kadrow BEZ przebudowy gry: edytujesz JSON, odpalasz exe ponownie (sekundy, nie minuty).
3. Sesja: wlasna kamera (Camera.CopyFrom + URP data), gracz SetActive(false) po
   sklonowaniu ustawien kamery, wszystkie Canvasy enabled=false PRZED KAZDYM kadrem
   (dymki world-space powstaja w trakcie), AudioListener.volume=0.
4. Pora dnia per kadr: TimeManager.SetHour + PauseTime(true) (slonce nie plynie).
5. `ScreenCapture.CaptureScreenshot(path, superSize:2)` przy oknie 1080p = 4K;
   plik pisze sie ASYNCHRONICZNIE - petla czekajaca na File.Exists z timeoutem.
6. Zrzut "landmarkow" (pozycje obiektow po slowach-kluczach nazw) do txt - sciaga
   do celowania kolejnych kadrow bez otwierania Edytora. Kluczowe przy celowaniu
   na slepo: pierwsza seria to rozpoznanie, druga wlasciwa.
7. Opcjonalnie `-photosave=N`: wczytaj zapis gry (rozwiniety swiat zamiast pustego
   startu). BEZPIECZENSTWO: SetAutoSaveEnabled(false) ZANIM Load; ladowac KOPIE
   save'a gracza (plik skopiowany do wolnego slotu, po sesji skasowany).

## Gotchas
- Sloty save maja walidacje zakresu (0..maxSlots-1) - kopia musi trafic w wazny slot.
- Kadry "z akcji" (minigry, klienci) wymagaja wyrezyserowania stanu gry - sama
  teleportacja kamery nie wystarczy.
- Petla weryfikacji: agent OGLADA wyprodukowane PNG (multimodal) i poprawia JSON -
  dziala bez czlowieka w petli az do etapu ocen estetycznych.

## Walidacja
Kerf 2026-07-24: 4 sesje, 16 kadrow 4K, w tym wnetrza hali z NPC z wczytanego
save'a (poziom 8) i warianty pory dnia. Sonda build-smoke 225/225 po zmianach.
