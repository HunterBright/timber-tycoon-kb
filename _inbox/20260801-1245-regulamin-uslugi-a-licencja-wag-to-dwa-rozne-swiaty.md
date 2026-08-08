---
title: Regulamin uslugi w chmurze i licencja pobieranych wag to dwa rozne dokumenty - sprawdzaj oba
type: lesson
status: draft
confidence: high
verified: 2026-08-01
tags: [licencje, generatory-3d, ai-gen, prawo, weryfikacja, znaki-wodne, low-poly]
date: 2026-08-01
project: Kerf - Sawmill Tycoon
source: 'https://www.tencentcloud.com/document/product/301/9248 , https://intl.cloud.tencent.com/document/product/301/78149 (Tencent HY 3D Global: Terms of Service 2026.02.06, Acceptable Use Policy 2025.12.27, Privacy Policy 2025.12.27 + test 8 modeli 2026-08-01)'
applies_to: [kazdy generator assetow AI rozwazany do gry komercyjnej]
severity: high
time_lost: ok. 6 miesiecy odrzucania narzedzia, ktore bylo dostepne
---

# Regulamin uslugi w chmurze i licencja pobieranych wag to dwa rozne dokumenty

## Czego to kosztowalo

Przez kilka miesiecy mielismy Hunyuan3D zapisany jako **niedostepny w Unii Europejskiej**
i wracalismy do tej decyzji kilka razy. Bylo to prawda - ale tylko o **jednej z dwoch drog**.

Ten sam producent, ta sama technologia, dwa zupelnie rozne dokumenty prawne:

| Droga | Dokument | Unia Europejska |
|---|---|---|
| Pobrane wagi, uruchamiane lokalnie | licencja spolecznosciowa w repozytorium | **wykluczona wprost**, plus blokada pobierania na Hugging Face (`extra_gated_eu_disallowed`) |
| Ta sama technologia jako usluga w chmurze | regulamin uslugi | **osobny podmiot dla EOG** (holenderska spolka), prawo Anglii i Walii, pelna maszyneria RODO |

Producent nie tylko dopuszcza UE w chmurze - **zalozyl pod to europejska spolke**.

## Zasada

> Odrzucajac narzedzie z powodu licencji, sprawdz, czy nie istnieje **druga droga
> dostepu do tej samej technologii, rzadzona innym dokumentem**.

Praktycznie: licencja wag rzadzi tym, co pobierasz i uruchamiasz u siebie.
Regulamin uslugi rzadzi tym, co robisz na ich serwerze. Odrzucenie jednego
nie zamyka drugiego, a wnioski z jednego **nie przenosza sie** na drugie.

## Drugi wniosek: prawa do wyniku to nie to samo, co czysty plik

Przy assetach AI trzeba sprawdzic **dwie niezalezne rzeczy**, a my do tej pory
sprawdzalismy tylko pierwsza:

1. **Czy wolno uzyc wyniku komercyjnie** - to jest w licencji albo regulaminie.
2. **Czy w pliku nie ma cudzych oznaczen** - to jest osobne pytanie.

Regulamin HY 3D Global przenosi prawa do wyniku na uzytkownika (punkt 6.3),
a jednoczesnie zapowiada dodawanie do wyniku **"znakow wodnych widocznych golym okiem"**
(punkt 6.7), ktorych regulamin dopuszczalnego uzycia **zabrania usuwac** (punkt 13).

Te dwa zdania nie sa sprzeczne. "Model jest Twoj" i "w modelu siedzi nasze
oznaczenie, ktorego nie wolno Ci ruszyc" moga stac obok siebie w jednej umowie.

## Jak to sprawdzic - cztery kontrole, nie jedna

Test na osmiu modelach pobranych 2026-08-01 (`tools/sprawdz_znak_wodny.py`):

1. **Napisy w pliku.** Szukaj nazwy producenta, slowa "watermark", "AI generated",
   standardow oznaczania tresci (C2PA, content credentials).
2. **Osadzone tekstury.** W FBX leza w srodku jako surowe bajty - wyciagnij je
   po naglowkach PNG/JPEG i **obejrzyj**. Zaden skrypt nie zobaczy napisu
   wypalonego w obrazku.
3. **Rogi tekstury w skali 1:1.** Pomniejszony podglad gubi maly znak wodny.
   Rogi to najczestsze miejsce.
4. **Liczba obiektow w modelu.** Znak wodny bywa **osobna siatka** doklejona
   do sceny. Model postaci ma miec jeden obiekt.

**Wynik testu:** wszystkie cztery kontrole czyste. Zero oznaczen. Zapowiedz
z regulaminu **nie jest dzis stosowana do modeli 3D** - ale zostaje w umowie,
a regulamin wolno zmienic ze skutkiem natychmiastowym (punkt 12.3).
Wynik pobrany dzis zostaje czysty; o jutrzejszym nic nie wiadomo.

## Rzecz, ktora przy okazji wyszla

Metadane pobranych plikow: `Blender (stable FBX IO) - 4.2.3 LTS`. Producent
eksportuje przez Blendera, dokladnie tak jak my. Zadnej wlasnej sygnatury.

Ustawienie "50k" daje **dokladnie 50 000 scianek i 100% trojkatow, zero czworokatow**.
Dla rigowania to ma znaczenie: czworokaty deformuja sie lepiej. Retopologia
zostaje osobnym krokiem, chyba ze producent ma tryb "wspiera topologie".

## Powiazane

- [[gate-must-have-provable-failure-mode]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: generatory-3d, licencje, low-poly
- [[20260808-1120-zasoby-w-dodatkach-blendera-maja-byc-cc0|Zasoby w dodatkach Blendera maja byc na CC0, a nie na GPL dodatku]] - wspolne: prawo, licencje
- [[20260801-1140-licencja-modelu-ai-to-trzy-osobne-dokumenty|Licencja modelu AI to trzy osobne dokumenty i wystarczy, ze jeden zabroni]] - wspolne: prawo, licencje
- [[20260801-0825-licencja-marki-nie-jest-licencja-produktu|Licencja marki nie jest licencja produktu]] - wspolne: generatory-3d, licencje
- [[20260725-0625-ai-model-community-license-excludes-eu|Nie stawiaj pipeline'u assetow na modelu AI z licencja "Community", nie czytajac pierwszej linii LICENSE]] - wspolne: prawo, licencje
<!-- /POWIAZANE:auto -->
