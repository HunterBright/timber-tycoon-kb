---
title: 'Pętla dźwiękowa: zmierz styk, przenikaj ogon w początek, dopisz próg do sondy'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-22'
project: Kerf - Sawmill Tycoon
tags:
- audio
- loop
- crossfade
- ffmpeg
- sfx
- measurement
- build-probe
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Pętla dźwiękowa: zmierz styk, przenikaj ogon w początek, dopisz próg do sondy

## When to use

Gracz mówi "słychać przeskok / dziurę / kliknięcie w zapętlonym dźwięku" (silnik, maszyna,
piła, wiatr). Dotyczy zwłaszcza materiału z generatorów AI (ElevenLabs, Suno) i próbek
wyciętych z dłuższych nagrań - jedne i drugie domyślnie mają rampę wejścia i wyjścia,
bo są robione jako dźwięk NA RAZ, nie jako pętla.

## Steps

1. **Zmierz, zanim cokolwiek zmienisz.** Głośność (RMS) pierwszych i ostatnich 50 ms:

   ```bash
   ffmpeg -v error -i loop.wav -f s16le -ac 1 -ar 48000 - | python policz_rms.py
   ```

   Zdrowa pętla ma oba końce w granicach ~±20%. Zmierzone wady w tym projekcie:
   silnik 25% (koniec 4x cichszy od początku), maszyna 28%. To jest ta "dziura".
   Drugi tryb wady: poziomy równe, ale ostatnia próbka daleko od zera - wtedy słychać
   trzask, nie dziurę.

2. **Odetnij rampy.** Policz obwiednię w oknach 10 ms, weź medianę, znajdź pierwsze i ostatnie
   okno powyżej ~55% mediany. To jest "mięso" nagrania bez wyciszeń.

3. **Przenikaj ogon w początek** (equal-power, 200-250 ms; dla klipów poniżej sekundy 60 ms):

   ```
   wynik[i] = ogon[i] * cos(t*pi/2) + poczatek[i] * sin(t*pi/2)   dla i < xf
   ```

   Długość wyniku = mięso minus długość przenikania. Ostatnia próbka przechodzi płynnie
   w pierwszą, bo pierwsze xf ms to już zmiksowany ogon.

4. **Zmierz ponownie i zapisz obie liczby.** W tym projekcie: 25% → 97%, 28% → 80%.

5. **Dopisz próg do sondy buildowej**, z progiem POMIĘDZY zmierzonym stanem zepsutym
   a naprawionym (tu: 0.55 przy zepsutym 0.22-0.28 i naprawionym 0.86-1.00).
   Czerwona próbka nie musi sięgać po stary plik (Unity go nie zapakuje, gdy przestanie
   być używany) - wystarczy, że próbka **wycisza ogon w pamięci do 25%**, czyli odtwarza
   zmierzoną wadę 1:1.

## Why this works

Ucho słyszy w miejscu zapętlenia dwie różne rzeczy: skok GŁOŚNOŚCI (dziura po rampie
wyjścia) i skok CIĄGŁOŚCI fali (trzask, gdy ostatnia próbka jest daleko od pierwszej).
Przenikanie ogona w początek załatwia oba naraz, bo w miejscu sklejenia obie strony
są tym samym sygnałem, tylko z odwrotnymi wagami. Krzywe cos/sin (equal-power) trzymają
stałą energię - liniowe dałyby słyszalne przygaszenie w połowie przenikania.

## Trade-offs

- Plik skraca się o długość przenikania. Przy krótkich klipach (0,3 s) to zauważalna część,
  więc przenikanie musi być proporcjonalnie krótsze.
- Dźwięki o dużej DYNAMICE (upadek drzewa: cichy trzask, potem huk) tracą na tym zabiegu -
  ale one nie są pętlami. Nie stosować kompresji "żeby dociągnąć głośność" do materiału,
  który ma być dynamiczny.
- W silniku dźwiękowym: pętla **nie może być trzymana jako skompresowana w pamięci**.
  Dekoduje się w kółko przez cały czas grania, potrafi dołożyć własną przerwę w miejscu
  zapętlenia, a w Unity `AudioClip.GetData` na takim klipie ODMAWIA - czyli sonda nie ma
  jak zmierzyć styku. Pętla = rozpakowana przy wczytaniu.

## Variants

- **Bardzo krótkie pętle** (poniżej 0,5 s): przenikanie 40-60 ms, inaczej zjada połowę klipu.
- **Materiał szumowy** (piła, wiatr): szum sam maskuje szew, więc przenikanie może być krótsze.
  Materiał TONALNY zdradza szew nawet przy równych poziomach - tam liczy się dopasowanie
  fazy, nie tylko głośności.
- **Gdy nie wolno ruszać pliku**: dwa źródła dźwięku grające naprzemiennie z zakładką w
  silniku. Działa, ale kosztuje drugi kanał i komplikuje sterowanie wysokością dźwięku.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[audio-asset-pipeline|Audio Asset Pipeline (ElevenLabs + Suno + FFmpeg)]] - wspolne: ffmpeg, audio
- [[ambient-crossfade-zone-based|Ambient Crossfade Zone-Based Pattern]] - wspolne: crossfade, audio
- [[20260625-0714-loop-fade-timefit-sfx-pattern|Dopasowanie SFX o stałej długości do akcji o zmiennej długości: pętla + wygaszenie]] - wspolne: sfx, audio
- [[footstep-raycast-surface-detection|Footstep Raycast Surface Detection]] - wspolne: sfx, audio
- [[20260722-1625-measure-before-fixing-serialization-hunch|Brakujący klucz w assecie NIE oznacza zera - zmierz, zanim "naprawisz"]] - wspolne: build-probe, measurement
- [[20260626-1016-unity-one-sided-audio-channel-balance|Dźwięk słychać tylko z jednej strony → najpierw sprawdź balans kanałów ŹRÓDŁA]] - wspolne: ffmpeg, audio
<!-- /POWIAZANE:auto -->
