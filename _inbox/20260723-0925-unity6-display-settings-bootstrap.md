---
title: 'Globalne ustawienia ekranu w Unity 6: natywna rozdzielczosc @ cap Hz od pierwszej klatki'
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-23'
project: Kerf - Sawmill Tycoon
tags:
- unity6
- display
- resolution
- refresh-rate
- fps-cap
- vsync
- settings
- bootstrap
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Globalne ustawienia ekranu w Unity 6: natywna rozdzielczosc @ cap Hz od pierwszej klatki

## Problem
Ustawienia grafiki trzymane w zapisie gry (per-save) aplikuja sie dopiero po wczytaniu
save'a - menu glowne i pierwszy start leca na domyslnych wartosciach silnika. Do tego
gra bez `Application.targetFrameRate` i z `vSyncCount 0` renderuje bez ogranicznika
(zmierzono ~886 FPS - grzanie GPU, artefakty zalezne od delta time).

## Wzorzec
1. **Wlasny plik JSON poza systemem save'ow** (`display_settings.json` w
   `Application.persistentDataPath`), czytany/zapisywany bezposrednio (JsonUtility + File).
2. **Hook `[RuntimeInitializeOnLoadMethod(BeforeSceneLoad)]`**: wczytaj plik; brak/niewalidny
   -> domyslne pierwszego startu; Apply; zapisz (pierwszy start staje sie deterministyczny).
3. **Domyslne pierwszego startu**: natywna pulpitu z `Display.main.systemWidth/Height`
   (NIE Screen.currentResolution - to biezace okno), odswiezanie = najwyzsze <= cap
   z `Screen.resolutions`, fpsLimit = min(cap, max Hz monitora).
4. **Drugi hook AfterSceneLoad** spawnuje jednorazowy MonoBehaviour, ktory po 2 klatkach
   porownuje stan i w razie rozjazdu aplikuje raz jeszcze (sterowniki ignoruja bardzo
   wczesne SetResolution; zlecenie i tak wykonuje sie "od nastepnej klatki").

## Kluczowe gotchas
- **`SetQualityLevel` po cichu nadpisuje `vSyncCount`** wartoscia zapisana w poziomie
  jakosci. Kazda zmiana jakosci MUSI byc potem doszlifowana: `vSyncCount = 0` +
  `targetFrameRate = limit` PO SetQualityLevel. Jedna funkcja Apply() z nosna kolejnoscia.
- **Cap na Hz dawaj z zapasem 0.5** (np. 144.5): monitory zglaszaja ulamkowe tryby
  (144001/1000 = 144.001 Hz; 143856/1000 = 143.856 Hz). Trzymaj RefreshRate jako ulamek
  (numerator/denominator), nie double - ponowny SetResolution jest bit-exact.
- **Wybrane Hz dziala tylko w ExclusiveFullScreen**. W FullScreenWindow (borderless)
  i Windowed o prezentacji decyduje pulpit - "max X Hz" realizuje targetFrameRate.
- **Boot-hook pomija Edytor** (`Application.isEditor`): Edytor dzieli persistentDataPath
  z buildem - wartosci z okna Game zasmiecilyby plik gracza. Skutek: logika dziala TYLKO
  w buildzie (rodzina "Edytor klamie") - weryfikacja wylacznie realnym buildem.
- **Lista poziomow jakosci jest przycinana per platforme w buildzie** - indeks poziomu
  w buildzie != indeks w Edytorze, jesli jakis poziom wyklucza platforme. Indeks zapisany
  w pliku jest spojny tylko w obrebie buildow.
- **Bezpiecznik zmiany trybu**: Apply bez zapisu na dysk + dialog z odliczaniem
  (czas REALNY - `realtimeSinceStartup`, bo menu czesto stoi na timeScale 0); zapis dopiero
  po potwierdzeniu; timeout = przywrocenie migawki. Crash przy czarnym ekranie nie utrwala
  zlego trybu - nastepny start czyta stary plik.
- **W trybie Windowed nie walcz o rozmiar okna** przy dociskaniu (gracz mogl je zmienic).

## Polityka VSync + limit FPS (jedna zasada)
VSync wlaczony = `vSyncCount 1` + `targetFrameRate -1` (monitor dyktuje tempo, limit
ignorowany i wyszarzony w UI); VSync wylaczony = `vSyncCount 0` + limit aktywny.
Dwa hamulce naraz (vsync + targetFrameRate) daja nierowne tempo klatek - nigdy oba.
Nowe pole bool w istniejacym JSON wczytuje sie przez JsonUtility jako false przy braku
w starym pliku = darmowa migracja bez resetu ustawien gracza.

## Kursor uwieziony w oknie (bonus wzorca)
Jesli caly projekt pisze Cursor.lockState w JEDNYM centralnym menedzerze (refcount
wlascicieli), zamiana None -> Confined w stanie "menu otwarte" uwiezi kursor w oknie
gry we wszystkich trybach ekranu jedna linijka (w Edytorze zostawic None - Confined
w oknie Game utrudnia prace). Confined liczy sie jako "odblokowany" przy porownaniach
`!= CursorLockMode.Locked`.

## Walidacja pliku miedzy sesjami
Zapisana rozdzielczosc musi istniec w `Screen.resolutions` LUB rownac sie natywnej pulpitu
(wymiana monitora, reczna edycja pliku) - inaczej odbuduj domyslne. Uwaga: przy zmianie
trybu okna sprawdzaj enum FullScreenMode wprost (0/1/3), MaximizedWindow (2) nie wystepuje
na Windows.
