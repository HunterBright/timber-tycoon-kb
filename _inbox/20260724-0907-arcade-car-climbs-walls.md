---
title: Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-24'
project: Kerf - Sawmill Tycoon
tags:
- unity
- physics
- rigidbody
- vehicle
- arcade-car
- raycast
- ground-check
- slope
applies_to: []
source: ''
severity: critical
suggested-category: engine/lessons
---

# Arkadowe auto na Rigidbody wspina sie po scianach i odlatuje w niebo

## Objaw

Auto sterowane silami (AddForce, bez WheelColliderow) po najechaniu na pionowa przeszkode
(slup latarni, sciana) wspina sie po niej i na trzymanym gazie jedzie pionowo w gore "do nieba".
Wyglada jak "brak grawitacji / przyklejanie do powierzchni", ale grawitacja jest wlaczona,
a zadnego kodu przyklejania nie ma - efekt jest w pelni emergentny.

## Przyczyna (trzy elementy MUSZA wystapic razem)

1. **Ciag wzdluz osi nadwozia.** `AddForce(transform.forward * gaz)` - os nadwozia przechyla sie
   razem z autem. Gdy nos zadrze sie na przeszkodzie, ciag dostaje skladowa pionowa. Przy sile
   napedu > grawitacja (typowe: 2-10x, bo acceleration musi pokonac drag) auto wygrywa
   z grawitacja juz przy kilkunastu-kilkudziesieciu stopniach przechylu. Limit predkosci przez
   clamp magnitude NIE pomaga - ogranicza szybkosc, nie kierunek.
2. **Naiwny ground-check.** Raycast w dol spod kol bez odczytu normalnej: "trafilem cokolwiek =
   jestem na ziemi". Slup pod przechylonym kolem tez sie liczy; wystarczy, ze czesc kol widzi
   teren, a gaz dziala dalej. Maska warstw NIE ratuje, gdy propsy leza na tej samej warstwie
   co grunt - odroznia je tylko NACHYLENIE (kat normalnej vs pion).
3. **Zerowe tarcie na colliderach auta** (celowe, zeby auto nie zacinalo sie o drobiazgi) -
   auto slizga sie w gore po przeszkodzie zamiast sie o nia zatrzymac.

## Naprawa (chirurgiczna, bez zmiany materialow i colliderow)

- **Dwie flagi, nie jedna**: `isGrounded` (kazde trafienie) dalej steruje skretem i tarciem
  bocznym - auto sterowne nawet zsuwajac sie ze stromizny. Nowa `isOnDrivableGround`
  (2+ kol nad powierzchnia o nachyleniu < prog, np. 50 st.) wlacza WYLACZNIE gaz.
  Wpakowanie filtru nachylenia do isGrounded odbiera sterownosc na zboczach - pulapka.
- **Rzut ciagu na plaszczyzne gruntu**: `ProjectOnPlane(fwd, usredniona normalna jezdnych
  trafien).normalized`. Na plaskim gaz poziomy niezaleznie od przechylu nadwozia; na rampie
  pcha pod gorke jak wczesniej. Normalizacja OBOWIAZKOWA - bez niej sila spada z cos(kata)
  i znika kalibracja predkosci maksymalnej. Rzut zdegenerowany -> zero sily, nigdy surowy fwd.
- **Znakowa bramka gazu**: kat osi ciagu (w kierunku jazdy) PONAD plaszczyzna gruntu > prog
  (np. 30 st.) -> gaz plynnie do zera (falloff ~10 st. przeciw oscylacjom). Znak ma znaczenie:
  ciag W DOL plaszczyzny (cofanie z przeszkody, zjazd, grzbiet wzgorza) nigdy nie jest blokowany.
  Kat WZGLEDEM plaszczyzny (nie absolutny pitch!) - na legalnej rampie nadwozie jest rownolegle
  do plaszczyzny, wiec bramka nie tnie niezaleznie od stromizny rampy.
- Progi z POMIARU tras, nie ze zgadywania (narzedzie raycastujace wzdluz waypointow/drog
  raportuje max kat normalnej i max spadek; prog = najostrzejsza legalna trasa + >=10 st.).

## Czego NIE robic

- NIE dodawac tarcia na colliderach auta (auto zacznie sie przyklejac do scian - gorzej).
- NIE dodawac "dodatkowej grawitacji" - blad jest w kierunku ciagu, nie w slabej grawitacji.
- NIE tlumic predkosci pionowej ani nie zmieniac clampu predkosci - psuje grzbiety wzgorz,
  zjazdy z polek i wjazd do wody.
- NIE wciskac filtru nachylenia do glownej flagi grounded (utrata sterownosci - patrz wyzej).

## Test wart utrwalenia

Czyste funkcje statyczne (filtr normalnej, rzut, bramka) + smoke-test na syntetycznych normalach
(0/40/60/90 st.) i na realnym colliderze slupa. Czerwona dzwignia: flaga cmdline przywracajaca
stan sprzed naprawy WEWNATRZ tych samych funkcji - bug realnie wraca w tym samym buildzie,
wiec dowod porazki testu jest uczciwy.
