---
title: 'Prop niesiony przez NPC: blokada pionu w LateUpdate + render osi propa zamiast zgadywania'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-17'
project: Kerf - Sawmill Tycoon
tags:
- unity
- npc
- animation
- prop-attachment
- lateupdate
- diagnostics
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Prop niesiony przez NPC: blokada pionu w LateUpdate + render osi propa zamiast zgadywania

ZWALIDOWANE przez playtest (2026-07-17): skrzynia kasjera otworem do góry we wszystkich fazach marszu.

## Problem
Prop (skrzynka, taca, wiadro) doczepiony jako dziecko kości dłoni dziedziczy PEŁNĄ rotację ręki -
w marszu kiwa się i przewraca razem z machającą dłonią. Strojenie stałej lokalnej rotacji pod jedną
pozę nie może działać: klip ma wiele faz, każda ustawia dłoń inaczej.

## Wzorzec: blokada pionu PO animatorze
- Pozycja: prop zostaje dzieckiem kości dłoni (podąża za ręką) ze strojoną stałą lokalną pozycją.
- Rotacja: w `LateUpdate` (MUSI być po animatorze, inaczej animator nadpisze) nadpisz rotację
  ŚWIATA: `prop.rotation = Quaternion.LookRotation(splaszczony_przod_ciala, Vector3.up) * KorektaOsi`.
  Efekt: prop zawsze wyprostowany, front propa = front ciała, zero kiwania, zero strojenia pod klipy.
- `KorektaOsi` = stały kwaternion mapujący konwencję modelu propa na pożądaną orientację
  (u nas otwór skrzyni leżał na lokalnym -Z, nie +Y -> korekta `Euler(90,0,0)`).

## Diagnostyka: render osi propa (nie zgaduj konwencji modelu)
Modele (zwłaszcza generowane/kupione) mają dowolnie zorientowane osie lokalne. Zamiast iterować
"na czuja": zainstancjonuj SAM prop w rotacji zerowej i wyrenderuj z 5 stron (góra, +Z, -Z, +X, izo).
Jedna seria zdjęć rozstrzyga definitywnie, na której ścianie jest otwór/front - stąd KorektaOsi
w jednej iteracji. Dodatkowo zrzut hierarchii z lokalnymi rotacjami dzieci (siatka może mieć
zapieczony obrót z FBX, który myli ocenę "po transformie").

## Strojenie pozycji: render 1:1 z grą, nie odpalanie gry
Sampluj klip noszenia przez PlayableGraph na prefabie NPC z propem doczepionym IDENTYCZNIE jak
w grze (ta sama blokada pionu) i renderuj kilka faz klipu z 3 ujęć (przód/bok/z góry). Uwaga na
układ odniesienia: prośba "wyżej" jest w PIONIE ŚWIATA, a stała pozycji w układzie przekrzywionej
dłoni - deltę świata trzeba przeliczyć iloczynem skalarnym z osiami dłoni w pozie klipu.

**Why:** każdy NPC niosący przedmiot w dowolnej grze ma ten problem; naiwne child+stała rotacja
wygląda dobrze w idle i psuje się w marszu, a zgadywanie osi propa pali iteracje builda.
**How to apply:** przy doczepianiu propa do kości: (1) render osi propa w rotacji zerowej,
(2) pozycja jako dziecko kości + rotacja świata w LateUpdate z korektą osi, (3) strojenie
pozycji przez sampling klipu offline, (4) werdykt wizualny gracza jako zamknięcie.
