---
title: ScreenCapture.CaptureScreenshotAsTexture w ciele korutyny lapie POPRZEDNIA klatke
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- unity
- screenshot
- screencapture
- coroutine
- ui
- verification
- probe
applies_to:
- unity-6
- unity-2021+
- any-unity-version
source: ''
severity: high
time_lost: ~1 build cycle (~8 min) + would have shipped an unverified UI
promoted: '2026-07-30'
---

# ScreenCapture.CaptureScreenshotAsTexture w ciele korutyny lapie POPRZEDNIA klatke

## Problem

Automat weryfikacyjny (sonda w prawdziwym buildzie) mial dowiesc, ze przebudowane okno UI
wyglada dobrze: buduje okno, mierzy geometrie, robi zrzut ekranu do PNG. Wszystkie pomiary
wyszly na zielono, PNG powstal i mial 1,6 MB (czyli NIE byl czarny ani pusty).

Na zrzucie bylo **zupelnie inne okno** - paragon klienta z wczesniejszej sekcji sondy, a nie
tablica, ktora wlasnie zmierzono. Gdyby nikt nie otworzyl pliku, "dowod wizualny" potwierdzalby
cudzy ekran.

## Root cause

`ScreenCapture.CaptureScreenshotAsTexture()` czyta **tylny bufor** (back buffer) karty graficznej.
W ciele korutyny (po `yield return null`) jestesmy w fazie Update NOWEJ klatki - ta klatka nie
zostala jeszcze narysowana, wiec w buforze siedzi obraz klatki POPRZEDNIEJ. API jest wazne
dopiero po `yield return new WaitForEndOfFrame()` (albo z `OnPostRender`).

Trzy rzeczy sprawily, ze blad byl cichy:
- rozmiar pliku wyglada normalnie (to prawdziwy, pelny zrzut - tylko nie ten, o ktory chodzilo),
- `try/catch` wokol zapisu nic nie lapie, bo nic nie rzuca wyjatkiem,
- jesli miedzy pomiarem a zrzutem stalo inne pelnoekranowe okno, obraz jest wiarygodny
  na pierwszy rzut oka.

## Solution

Zrzut MUSI isc po koncu klatki:

```csharp
// C# zabrania `yield return` w bloku try z catch - WaitForEndOfFrame zostaje poza try.
if (!Application.isBatchMode)          // w trybie wsadowym WaitForEndOfFrame nigdy nie wraca
{
    yield return new WaitForEndOfFrame();
    SaveShot("okno.png");              // dopiero teraz CaptureScreenshotAsTexture
}
```

Uwaga na tryb wsadowy: `WaitForEndOfFrame` w `-batchmode`/`-nographics` **nie wraca nigdy**,
wiec korutyna zawisa, a z nia caly automat. Bramka na `Application.isBatchMode` jest obowiazkowa,
jesli ten sam kod moze pojsc headless.

Alternatywa nie do zastosowania przy overlay UI: renderowanie kamery do `RenderTexture` jest
niezalezne od fazy klatki, ale kamera **nigdy nie widzi** kanwy w trybie `ScreenSpaceOverlay`.
Dla UI zostaje sciezka z `WaitForEndOfFrame`.

## What didn't work

- **Patrzenie na rozmiar/istnienie pliku jako dowod.** 1,6 MB PNG to nie dowod, ze zrzut jest
  z wlasciwego momentu.
- **Zaufanie zielonym pomiarom.** Pomiary geometrii byly poprawne i niezalezne od zrzutu - to
  wlasnie uspilo czujnosc. Liczby moga byc prawdziwe, a obrazek i tak z innej klatki.
- **`try/catch` jako siatka bezpieczenstwa.** Tu nie ma wyjatku, wiec catch nigdy nie zadziala.

## Transferability

Dotyczy kazdego projektu Unity, ktory robi zrzuty ekranu z kodu: automatyczne dowody wizualne
w CI, generowanie materialow marketingowych, zrzuty "przed/po" przy zmianach UI, fotogaleria
w grze. Nie zalezy od gatunku, renderera ani wersji Unity.

Szersza lekcja, wazniejsza niz samo API: **kazdy artefakt dowodowy musi byc obejrzany co najmniej
raz przez cos, co potrafi go zweryfikowac.** Automat, ktory produkuje "dowod" i nigdy go nie
sprawdza, produkuje falszywe poczucie bezpieczenstwa - i to tym skuteczniej, im bardziej
wiarygodnie dowod wyglada.

## Related
- [[gate-must-have-provable-failure-mode]]
- [[20260721-1215-ui-fit-check-measuring-rect-instead-of-text]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260718-0805-headless-visual-proof-batchmode|Dowod wizualny z Unity batchmode (bez otwierania Edytora)]] - wspolne: screenshot, verification
- [[20260721-1215-ui-fit-check-measuring-rect-instead-of-text|Sprawdzanie "czy tekst sie miesci" przez pomiar RectTransform]] - wspolne: probe, ui
<!-- /POWIAZANE:auto -->
