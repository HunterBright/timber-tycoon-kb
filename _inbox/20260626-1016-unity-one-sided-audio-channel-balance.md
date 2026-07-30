---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, audio, stereo, mono, force-to-mono, audioclip, ffmpeg, diagnosis]
date: 2026-06-26
status: draft
---

# Dźwięk słychać tylko z jednej strony → najpierw sprawdź balans kanałów ŹRÓDŁA

## Objaw
Konkretny dźwięk (tu: silnik auta) słychać tylko z lewego głośnika. Reszta gry OK.

## Pułapka diagnostyczna — kolejność eliminacji
Naturalny pierwszy strzał to „panorama / orientacja słuchacza" — ale to często NIE to. Eliminuj w tej kolejności:
1. **panStereo** na AudioSource (≠0 = bias). Sprawdź kod. (U nas: wszędzie 0.)
2. **spatialBlend + orientacja AudioListener.** Dźwięk 3D panuje wg osi `listener.transform.right`. Jeśli listener siedzi na obiekcie o przekręconej orientacji (np. auto z forward=`-transform.right`), panorama 3D się skrzywia. ALE: sprawdź NAPRAWDĘ, gdzie jest listener — u nas był na KAMERZE (ustawianej `LookAt`, orientacja poprawna), więc ta teoria PADŁA mimo że brzmiała przekonująco. Nie zakładaj — zweryfikuj GameObject słuchacza w żywym edytorze.
3. **Balans kanałów samego PLIKU.** Klip stereo z CISZĄ w jednym kanale, grany jako 2D, wiernie odtwarza L→lewy, R→prawy → słychać z jednej strony. To była przyczyna.

Kluczowe: dźwięk **2D** jednostronny ⇒ to prawie na pewno plik (2D nie panuje wg pozycji). Dźwięk **3D** jednostronny ⇒ pozycja/orientacja listenera.

## Jak ZMIERZYĆ balans kanałów (nie da się z .meta!)
Unity AudioImporter (.meta) pokazuje tylko `forceToMono`, NIE liczbę kanałów ani balans. Zmierz energię per-kanał:
- **Editor script przez MCP** (Coplay `execute_script`): `AudioClip.GetData(float[], 0)`, próbki interleaved [L,R,L,R...], policz RMS L vs R. Flaguj gdy `min(RMS_L,RMS_R)/max < 0.05`.
- GOTCHA: `GetData` zwraca false / zera dla klipów `loadType=CompressedInMemory`. Dla nich albo wymuś chwilowo DecompressOnLoad, albo **sparsuj plik WAV z dysku** (nagłówek fmt → channels/bits, chunk data → próbki PCM 16/24/32). MP3 skompresowany = trzeba dekodera.
- Skanuj `AssetDatabase.FindAssets("t:AudioClip")` — audyt wszystkich klipów naraz odpowiada „czy inne też są jednokanałowe".

## Skąd się bierze cichy kanał
Często z pipeline'u przetwarzania, nie z nagrania. U nas warianty `_Loud` powstały z **normalizacji ffmpeg** (rundy wcześniej) i wyszły z wyzerowanym prawym kanałem. Zawsze weryfikuj kanały PO obróbce ffmpeg/normalizacji.

## Fix
**Import → Force To Mono = true** (`AudioImporter.forceToMono`, potem `SaveAndReimport`). Zlewa do mono → Unity gra centralnie (oba głośniki) dla 2D, poprawnie spatializuje dla 3D. Zalety: odwracalne, NIE rusza pliku źródłowego, mniej pamięci, a dla SFX (silnik, kroki, uderzenia) mono i tak jest standardem. Gdy jeden kanał jest ciszą — nic nie tracisz, a forceToMono z normalizacją potrafi nawet podbić poziom. Weryfikacja po fixie: `AudioClip.channels == 1`.

## Anti-pattern
Rzucić się na „panorama/listener" i stroić spatialBlend/panStereo, gdy plik źródłowy ma martwy kanał. Najpierw zmierz balans kanałów klipu.
