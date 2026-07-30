---
title: Aktualizacja major Unity → wyjście z Safe Mode + sprzątanie deprecacji
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-06-26'
project: Kerf - Sawmill Tycoon
tags:
- unity
- upgrade
- deprecation
- safe-mode
- api-migration
- cs0619
- cs0618
- textmeshpro
- find-objects
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Aktualizacja major Unity → wyjście z Safe Mode + sprzątanie deprecacji

## Kontekst
Upgrade Unity `6000.3.5f1` → `6000.5.1f1` (Unity 6.5). Projekt wpadł w Safe Mode, w konsoli ściana ~150 linii „obsolete". Panika niepotrzebna — większość to ostrzeżenia.

## Lekcja kluczowa: triage błąd vs ostrzeżenie ZANIM cokolwiek ruszysz
- **`CS0619`** = obsolete-as-**ERROR** → TO blokuje kompilację i wyrzuca do Safe Mode. Tylko te trzeba naprawić, żeby ruszyć.
- **`CS0618`** = obsolete-as-**WARNING** → NIE blokuje. Można zostawić, ale w kolejnym major Unity stanie się `CS0619`.
- Sprawdź, czy projekt nie ma „warnings-as-errors" (brak `csc.rsp`/`mcs.rsp`, brak flagi w `.asmdef`). Jeśli nie ma — ostrzeżenia są nieblokujące. To pierwszy fakt do ustalenia.
- Praktyka: z 150 linii logu realnie blokowało ~11 (jeden nieużywany pakiet + 1 linia własnego kodu).

## Mapa typowych deprecacji Unity 6.5 (replacement 1:1, zachowanie bez zmian)
- `FindFirstObjectByType<T>()` → `FindAnyObjectByType<T>()` — **zachowaj** argument `FindObjectsInactive.X` jeśli jest. Dla singletonów/managerów „first" vs „any" nie robi różnicy (Unity samo rekomenduje).
- `FindObjectsByType<T>(FindObjectsSortMode.None)` → `FindObjectsByType<T>()`
- `FindObjectsByType<T>(FindObjectsInactive.X, FindObjectsSortMode.None)` → `FindObjectsByType<T>(FindObjectsInactive.X)` — **usuwasz TYLKO sort mode, filtr inactive zostaje**. To najczęstszy błąd: zgubienie filtra zmienia zachowanie (czy nieaktywne obiekty wchodzą do wyniku).
- TMP: `text.enableWordWrapping = true/false` → `text.textWrappingMode = TextWrappingModes.Normal/NoWrap` (enum w `TMPro`, namespace już jest skoro używasz TMP_Text).
- `UnityEngine.Object.GetInstanceID()` → `GetEntityId()` (zwraca `EntityId`, nie `int`). Gdy potrzebujesz `int` (np. ziarno do hasha) — **`GetHashCode()`**: dla `UnityEngine.Object` zwraca dokładnie stary instance id, jest `int`, nie jest deprecated. Najprostsza zamiana bez zmiany zachowania.
- `AssetDatabase.GetAssetPath(int)` / `EditorUtility.InstanceIDToObject(int)` → przeciążenia z `EntityId`. UWAGA: `GetAssetPath(Object)` (przyjmujące obiekt, nie int) NIE jest przestarzałe — nie myl wariantów, większość wywołań w grze przekazuje obiekt i są OK.

## Pakiet third-party rzuca CS0619 ze SWOJEGO kodu
- Błąd był w `Library/PackageCache/com.unity.polybrush/...` — nie w naszym kodzie. Źródła pakietu nie da się czysto poprawić (PackageCache nadpisywany przy resolve).
- **Reguła: jeśli pakiet jest nieużywany → USUŃ go** (jedna linia w `Packages/manifest.json`), nie łataj/embeduj. Weryfikacja nieużywania: grep po komponentach pakietu w scenach/prefabach/.asset + odwołaniach w kodzie. Polybrush (edytorowy painter) bywa zaległy w projektach robiących teren w Blenderze.
- Po usunięciu pakietu skasuj też jego osierocony folder danych (`Assets/<Pakiet> Data/`) — inaczej `.asset` z typami pakietu zostają jako „missing".

## Workflow migracji (szybki i pewny dla setek miejsc)
1. **Inwentaryzacja** dokładnych sygnatur PRZED edycją (rozróżnij warianty argumentów — patrz pułapka inactive vs sort mode).
2. **Batch przez `sed`** (token-precyzyjnie, bezpieczne dla UTF-8/BOM/CRLF — GNU sed przepuszcza bajty nietknięte). Edytuj tylko pliki z trafieniem: `grep -rlZ --include=*.cs 'wzorzec' Assets/ | xargs -0 -r sed -i '...'`. LLM-owe edytowanie 80 plików jest gorsze niż deterministyczny skrypt dla czysto mechanicznej podmiany.
3. **Dowód kompletności = re-grep do zera.** Po podmianie `grep` na stary token musi dać 0 trafień. To mocniejsza weryfikacja niż „policzyłem że zrobiłem N".
4. **Compile check** na końcu (tu: Coplay MCP `check_compile_errors` → „No compile errors”). Mostek MCP potrafi odpowiadać nawet w/po Safe Mode, gdy assembly się skompilują.

## Anti-pattern
Rzucanie się na całą ścianę „obsolete" naraz i ręczne klikanie po plikach. Najpierw oddziel błędy od ostrzeżeń, ustal że ostrzeżenia nie blokują, napraw blokery, a sprzątanie ostrzeżeń zrób masowo skryptem z weryfikacją re-grep.
