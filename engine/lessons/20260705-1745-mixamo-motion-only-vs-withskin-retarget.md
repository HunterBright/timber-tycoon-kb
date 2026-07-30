---
title: Mixamo "Without Skin" (motion-only) FBX psuje retarget Humanoid - uzyj "With Skin"
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-05'
project: Kerf - Sawmill Tycoon
tags:
- unity
- mixamo
- animation
- humanoid
- retargeting
- avatar
- fbx
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Mixamo "Without Skin" (motion-only) FBX psuje retarget Humanoid - uzyj "With Skin"

## Objaw
Animacja Mixamo nalozona na postac Humanoid retargetuje sie z wykrzywiona jedna strona ciala
(np. lewa noga strasznie wygieta, prawa OK; rece na roznej wysokosci). Chod bazowy i inne klipy
na tej samej postaci wygladaja poprawnie - problem jest specyficzny dla tego jednego klipu,
nie dla szkieletu postaci docelowej.

## Root cause
Plik pobrany z Mixamo w opcji "Without Skin" (albo eksport motion-only / Retargeted Clip) NIE
zawiera siatki ani czystej pozy bazowej (T-pozy). Gdy Unity importuje go jako Humanoid z avatarem
"Create From This Model", nie ma z czego poprawnie zbudowac mapowania miesni - zgaduje T-poze z
pierwszej klatki animacji i myli sie, zwykle asymetrycznie.

## Fix
Pobrac ten sam ruch z Mixamo w opcji "With Skin" (pelny FBX z postacia). Ma czysta T-poze w bind
pose -> Unity buduje poprawny avatar -> retarget czysty. Import jako Humanoid, Create From This
Model, loopTime=true.

## Co NIE zadzialalo
avatarSetup = CopyFromOther wskazujacy na avatar innego (dzialajacego) klipu Mixamo tego samego
szkieletu - nie naprawilo dystorsji.

## Diagnostyka (klip vs rig postaci docelowej)
Odtworz na tej samej postaci: (a) bazowy chod / inny znany-dobry klip, (b) podejrzany klip. Jesli
(a) czysty a (b) wykrzywiony -> wina klipu/importu. Jesli oba zle -> rig postaci docelowej.

## Sygnal po rozmiarze pliku
Motion-only FBX jest maly (setki KB, brak mesha). "With Skin" jest wiekszy (siatka + skinning +
czesto tekstury embedded). Ten sam ruch, 1.5x+ roznicy = pobrales motion-only.
