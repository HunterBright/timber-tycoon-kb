---
title: Dźwięk 3D jest za cichy nie przez głośność, tylko przez odległość kamery (AudioListener) od źródła
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-06-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- audio
- audiosource
- spatialblend
- 3d-sound
- rolloff
- audiolistener
applies_to:
- unity
source: ''
severity: medium
time_lost: ~2 iteracje play-test/diagnoza
promoted: '2026-07-30'
---

# Dźwięk 3D jest za cichy nie przez głośność, tylko przez odległość kamery (AudioListener) od źródła

## Problem
Dźwięk silnika pojazdu (i osobno dźwięk pracy maszyny w minigrze z kamerą widoku z góry) był ledwo słyszalny - cichszy niż kroki gracza. Podniesienie `volume` w bazie dźwięków z 0.4 na 0.85 prawie nic nie dało. Subiektywnie „auto za cicho", „maszyna nie ma dźwięku pracy".

## Root cause
Źródła grały jako **3D** (`AudioSource.spatialBlend = 1`) na pozycji obiektu (auta / maszyny). Tłumienie odległością liczone jest względem **AudioListener** (na aktywnej kamerze), a kamera była daleko od źródła:
- pojazd: kamera pościgowa 4-15 m za autem,
- minigra maszyny: kamera na widoku z góry, kilka metrów od maszyny.

Pula `AudioSource` była tworzona BEZ ustawiania `minDistance`/`maxDistance`/`rolloffMode`, więc obowiązywały domyślne Unity: `minDistance = 1`, `rolloffMode = Logarithmic`. Logarytmiczny rolloff to z grubsza `głośność ≈ minDistance / odległość`, czyli przy 8-10 m źródło spada do ~10-15% głośności. Pole `volume` jest mnożone PO tym tłumieniu, więc podbijanie liczby w bazie ledwo pomaga. Kroki brzmiały głośno, bo w trybie pieszym listener (kamera FPP) jest ~1 m od gracza.

## Solution
Dla dźwięków, które grają **tylko wtedy, gdy gracz jest „przy" źródle** (kamera i tak za nim podąża: własny pojazd, maszyna w minigrze) - zrobić je **2D** (`spatialBlend = 0`). Wtedy `volume` działa wprost, bez tłumienia odległością. Brak realnej wady, bo taki dźwięk i tak nie gra, gdy gracza tam nie ma, więc przestrzenność niczego nie wnosi.
- Pętla/one-shot przez warstwę audio: nie podawaj pozycji (u nas `PlayLoop(id)` bez `position` ⇒ 2D), albo ustaw `source.spatialBlend = 0` po starcie.
- Modulację pitcha (np. obroty silnika) zostaw - działa identycznie w 2D.

Alternatywa, gdy przestrzenność JEST potrzebna (np. źródło, koło którego gracz chodzi pieszo): zostaw 3D, ale ustaw `minDistance` ≥ typowa odległość kamery i/lub `rolloffMode = Linear` z sensownym `maxDistance` (tak robi u nas ambient wodospadu). To wymaga jawnego konfigurowania źródeł - domyślne minDistance=1 + Logarithmic jest pułapką.

## Druga warstwa: nieużyty headroom KLIPU (gdy 2D nadal za cicho)
Po naprawie na 2D dźwięk pojazdu DALEJ był cichszy od kroków. Pomiar `ffmpeg -i clip -af volumedetect -f null -` ujawnił, że plik silnika miał `max_volume = -16.5 dB` (peak ~0.15) - ~16 dB nieużytego zapasu. Unity `AudioSource.volume` jest w [0,1] i tylko TŁUMI względem poziomu nagrania; cichego pliku nie podbije się liczbą powyżej tego, co nagrane. Naprawa: znormalizować/wzmocnić sam PLIK (`ffmpeg -af "volume=+15.5dB"`, lub `loudnorm`) do ~ -1 dBFS peak, zaimportować, przepiąć - wtedy `volume` operuje na głośnym sygnale.
Dodatkowy trik diagnostyczny: w Unity `AudioClip.GetData` + peak/RMS porównaj podejrzany klip z takim, który NA PEWNO słychać. Jeśli „cichy" klip ma podobne/wyższe RMS, a i tak go nie słychać → problem to „nie gra"/maskowanie, nie głośność (kieruje śledztwo w inną stronę).

## What didn't work
- Podnoszenie samego `volume` w bazie dźwięków (0.4 → 0.85) - bez efektu: najpierw bo tłumienie odległością (3D), a po przejściu na 2D bo klip miał -16.5 dB peak (sufit nagrania).
- Założenie, że „skoro klip jest dobry i głośność wysoka, to będzie słychać" - pomija i geometrię listener↔source, i headroom samego pliku.

## Transferability
Czysto silnikowa pułapka Unity, niezależna od gatunku gry: każdy projekt z dźwiękiem 3D i kamerą oddaloną od źródła (pojazdy, kamera TPP/pościgowa, kamera minigry/cutscenki, RTS z odsuniętą kamerą) trafi na to samo. Reguła diagnostyczna: gdy dźwięk 3D jest „za cichy mimo wysokiej głośności" - najpierw sprawdź odległość AudioListener od źródła oraz `spatialBlend`/`rolloffMode`/`minDistance`, a nie samą głośność.

## Related
- (pula AudioSource bez ustawionych dystansów = ukryte domyślne Logarithmic/minDistance=1)
