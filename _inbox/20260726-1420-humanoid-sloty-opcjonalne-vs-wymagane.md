---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, humanoid, avatar, rig, bones, getbonetransform, mixamo, gate, false-green]
severity: medium
time_lost: "~25 min analizy (wylapane PRZED implementacja)"
date: 2026-07-26
status: draft
applies_to: [unity, humanoid-rig]
---

# Humanoid: sloty OPCJONALNE zwracaja null na poprawnym awatarze - fallback po nazwach nie moze byc pod jednym `!isHuman`

## Problem

Sprawdzenie w sondzie buildu szukalo kosci nog wzorcem:

```csharp
if (animator.isHuman) { /* GetBoneTransform dla 6 kosci */ }
// fallback po nazwach TYLKO gdy powyzsze nic nie dalo
```

Z planowanym rigiem (awatar humanoidalny, nazwy Mixamo, kosci `ToeBase`
obecne w szkielecie ale **celowo niepodpiete** do slotow `LeftToes`/`RightToes`)
ten uklad daje trwala slepote: `isHuman` jest prawdziwe, wiec fallback po
nazwach sie nie odpala, a `GetBoneTransform(HumanBodyBones.LeftToes)` zwraca
null, bo slot jest niezmapowany. Kosc **lezy obok, ma nazwe**, i nigdy nie
zostanie uzyta. Sprawdzenie przy kazdym buildzie zglaszalo braki palcow.

## Root cause

`HumanBodyBones` dzieli sie na sloty **wymagane** i **opcjonalne**. Avatar
przechodzi walidacje (`isValid`, `isHuman`) z niewypelnionymi slotami
opcjonalnymi - `Toes`, `Neck`, palce dloni, oczy, szczeka. Pusty slot
opcjonalny to **legalny stan poprawnego awatara**, a nie awaria.

Wniosek: `isHuman == true` nie znaczy "wszystkie kosci sa dostepne przez
mapowanie". Jeden warunek `!isHuman` na calym fallbacku miesza dwa rozne
przypadki:

- pusty slot **wymagany** (LowerLeg, Foot) = awatar naprawde zepsuty,
  i zaklejanie tego szukaniem po nazwach UKRYWA prawdziwa awarie,
- pusty slot **opcjonalny** (Toes) = normalny stan, kosc trzeba znalezc
  inaczej.

## Solution

Rozdzielic fallback po roli slotu, nie po jednej fladze awatara:

```csharp
// Golen i stopa: WYMAGANE. Nazwy tylko gdy awatar pusty albo Generic.
// Pusty slot na humanoidzie -> FAIL z nazwa kosci (nie zaklejac!).
if (!human) { /* dopasowanie nazw goleni i stopy */ }

// Palce: OPCJONALNE. Nazwy ZAWSZE, gdy slot wrocil pusty - takze na humanoidzie.
if (toeL == null && NameEndsWithAny(n, ToeNamesL)) toeL = t;
```

Dodatkowo, gdy kosc opcjonalna sluzy do POMIARU:

- brak takiej kosci nie moze oblewac bramki, ale **nie moze tez cicho
  podstawiac zastepnika** i udawac, ze liczba jest ta sama. Kat kostki liczony
  z odcinka kostka-palce i kat liczony z wlasnej osi stopy to dwie rozne
  metryki. Trzeba oznaczyc wynik jako niezmierzony i powiedziec w raporcie,
  ze liczby nie sa porownywalne,
- metryka, ktorej nie zmierzono, nie moze zostac zaliczona jako spelnione
  kryterium. Zero z braku pomiaru wyglada jak idealna symetria i daje
  **falszywy zielony** - trzeba je wykluczyc z werdyktu jawnie.

Dopasowanie nazw: rozne rigi w jednym projekcie zyja obok siebie (Mixamo
`mixamorig:LeftToeBase` i Blender `Toe.L`), wiec lista kandydatow na sufiks
plus porownanie bez rozroznienia wielkosci liter (`OrdinalIgnoreCase`).

## What didn't work

Pierwotny plan mowil "fallback po nazwach tylko gdy awatar jest null albo nie
isHuman" - regula spojna i zla, bo dla docelowego rigu (humanoid + niepodpiete
palce) nigdy by sie nie uruchomila. Wylapane analiza kodu przed implementacja,
nie pomiarem, bo rig jeszcze nie istnial.

## Transferability

Dotyczy kazdego projektu Unity z awatarami humanoidalnymi, zwlaszcza gdy w
jednym projekcie zyje kilka konwencji nazw kosci (asset store + Mixamo +
wlasny rig z Blendera). Szerzej: wzorzec "jedna flaga waznosci na cala grupe
zasobow" psuje sie zawsze, gdy grupa ma czlonkow wymaganych i opcjonalnych.

## Related
- [[discriminating-clip-vs-rig-vs-skin-humanoid-defect]]
- [[sonda-musi-umiec-zawiesc]]
