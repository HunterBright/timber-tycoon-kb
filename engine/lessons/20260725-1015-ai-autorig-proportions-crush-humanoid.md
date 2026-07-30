---
title: 'Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke'
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- humanoid
- rig
- ai-assets
- retargeting
- blender
- sonda
applies_to:
- unity-6
- humanoid-avatar
- ai-generated-characters
source: ''
severity: high
time_lost: ~3 h
promoted: '2026-07-30'
---

# Szkielet z auto-rigu AI ma inne proporcje niz siatka: postac w grze skladasie w harmonijke

## Problem
Model postaci wygenerowany przez AI (siatka + auto-rig w jednym potoku) wygladal poprawnie
w Blenderze i w podgladzie Unity, mial poprawny awatar humanoidalny (22 zmapowane kosci),
stal na ziemi. Po nalozeniu ZWYKLEJ animacji chodzenia z biblioteki projektu postac w grze
kurczyla sie do **61% wzrostu**: glowa normalna, tulow zgnieciony, nog praktycznie nie widac.
W pozie spoczynkowej (bind pose) wszystko bylo w porzadku - blad pojawial sie DOPIERO w ruchu.

## Root cause
Auto-rig wstawil kosci pasujace do INNYCH proporcji niz siatka:

| Punkt | Szkielet z auto-rigu | Wartosc oczekiwana |
|---|---|---|
| Staw biodrowy | 36% wzrostu | ~50% |
| Nasada glowy | 69% wzrostu | ~85% |

W bind pose to niewidoczne, bo skora jest przypieta do kosci tam, gdzie one sa. Ale silnik
retargetuje animacje humanoidalna wedlug proporcji AWATARA - ustawia szkielet po swojemu,
a skora idzie za nim i postac sie sklada.

## Solution
Zbudowac wlasny szkielet w Blenderze, mierzac SIATKE, zamiast ufac auto-rigowi:

1. Profil sylwetki: dla ~35 plastrow wzdluz osi pionowej policzyc szerokosc, glebokosc
   i liczbe wierzcholkow. Z tego czyta sie landmarki: buty (duza glebokosc przy ziemi),
   kostka (glebokosc spada z buta do lydki), nogawki, pas ramion (maksymalny zasieg w poziomie
   przy T-pozie), szyja (najwezszy przekroj nad ramionami), glowa.
2. Kosci ze STANDARDOWYMI nazwami Unity (Hips, Spine, Chest, Neck, Head, LeftUpperArm, ...) -
   auto-mapowanie awatara wtedy po prostu dziala.
3. Skorowanie: `parent_set(type='ARMATURE_AUTO')`, potem sprawdzic ZERO wierzcholkow bez wag.
4. Eksport FBX: `bake_space_transform=True` + `apply_scale_options='FBX_SCALE_ALL'`.

Efekt: wzrost w ruchu 98% wzrostu spoczynkowego, stopy 0,1 cm od gruntu.

## What didn't work
- **Detekcja krocza przez "wierzcholki przy osi"**: nogawki tej postaci prawie sie stykaja,
  wiec wykrywacz zwracal 25% wzrostu zamiast 34%. Trzeba bylo wypisac profil sylwetki
  i odczytac progi z liczb, a nie zgadywac.
- **`bakeAxisConversion` w importerze Unity**: normalizuje wezel siatki, ale wezel SZKIELETU
  i tak zostaje z obrotem 90 stopni. Nie to bylo przyczyna.
- **Reeksport z innym przelicznikiem skali**: naprawil osobny blad (zapadanie 72 cm pod ziemie),
  ale zgniatania nie ruszyl.

## GOTCHA: Unity trzyma stare mapowanie kosci w .meta
Po WYMIANIE szkieletu w tym samym pliku FBX import wywala sie na
`Transform 'Hip' for human bone 'Hips' not found` - silnik probuje nalozyc mapowanie
z poprzedniego rigu. Trzeba wyczyscic `humanDescription` (puste tablice `human` i `skeleton`)
i ustawic `autoGenerateAvatarMappingIfUnspecified = true` PRZED reimportem.

## GOTCHA: pudelko renderera klamie przy siatce ze szkieletem
`SkinnedMeshRenderer.bounds` niesie zapas na animacje (u nas 19 cm ponizej stop) i nie nadaje
sie do walidacji "stopy na zerze". Do pomiaru brac `sharedMesh.bounds` (po zapieczeniu obrotu
jest w metrach i w orientacji silnika) albo pozycje kosci stopy z awatara.

## Transferability
Kazdy projekt wpuszczajacy postacie z generatorow AI (Meshy, Tripo, Hunyuan, Unity AI).
Zasada ogolna: **auto-rig weryfikuj POMIAREM W RUCHU, nie w pozie spoczynkowej**. Najtanszy
sprawdzian: zmierzyc wysokosc obrysu postaci po 1-2 s odtwarzania animacji i porownac
z wysokoscia w bind pose. Ponizej ~80% = szkielet nie pasuje do siatki. Ten jeden pomiar
zlapalby caly problem w 30 sekund zamiast w 3 godziny.

## Related
- [[20260725-0625-ai-model-community-license-excludes-eu]] (ten sam potok assetow, strona prawna)
