---
title: CloudRig daje czysty rig do gry, bo odwraca domyslne use_deform Blendera
type: lesson
status: draft
confidence: medium
verified: '2026-08-11'
date: 2026-08-11
project: GameDevOS
tags:
- blender
- rigging
- unity
- fbx
- low-poly
- czworonogi
applies_to:
- kazdy projekt eksportujacy zrigowana postac z Blendera do Unity
- postacie nieludzkie, dla ktorych nie ma gotowego szablonu
source: 'https://extensions.blender.org/add-ons/cloudrig/ oraz https://projects.blender.org/Mets/CloudRig'
severity: medium
suggested-category: pipeline/lessons
time_lost: ''
---

# CloudRig daje czysty rig do gry, bo odwraca domyslne use_deform Blendera

## Problem

Rigify, standardowy generator rigow w Blenderze, robi rigi **filmowe, nie
growe**. Zmierzone u nas 09.08.2026 na szablonie wilka z zestawu zwierzat:

- szablon ma 190 kosci
- wygenerowany rig ma **823 kosci**
- deformuje z nich **197**
- w rigu siedzi pelny rig twarzy odziedziczony po szablonie ludzkim: brwi,
  powieki, jezyk, zeby, wargi, a takze lancuchy palcow i kosci piersi

**Eksport FBX wypuszcza wszystkie 823 kosci**, jesli nie odsiac ich po fladze
`use_deform`. Dla postaci ludzkiej to uciazliwe. Dla tlumu potworow w grze to
jest po prostu nie do przyjecia.

Naturalny odruch brzmi: napisac skrypt, ktory po generacji przejdzie po rigu
i pousuwa kosci sterujace. To dziala, ale jest to **leczenie objawu**: lista
tego, co usunac, zalezy od wersji szablonu i rozjezdza sie po cichu.

## Root cause

Rigify buduje kosci sterujace i deformujace w jednym drzewie i **zostawia
domyslne ustawienie Blendera**, w ktorym nowo utworzona kosc ma
`use_deform = True`. Flaga jest wiec wlaczona z rozpedu, a nie z decyzji.
Kazda kosc pomocnicza, ktora ktos dopisal do szablonu, domyslnie wchodzi
do eksportu.

CloudRig, generator rigow rozwijany przez Blender Studio, robi odwrotnie.
Komentarz w jego kodzie mowi to wprost (`rig_component_features/bone_info.py`,
okolica linii 311 - **odczytane przez zwiad, przeze mnie nie zweryfikowane
ponownie, bo serwer kodu Blendera odmowil pobrania surowego pliku**):

> `# The default value of use_deform in Blender is True, but for CloudRig,`
> `# False makes a LOT more sense.`

Czyli **kazda kosc rodzi sie jako niedeformujaca, a flage dostaja wylacznie
te celowo oznaczone**. Skutek jest mierzalny i jednoznaczny.

## Solution

Zmierzone 11.08.2026 w bezokiennym Blenderze 5.2:

| Pomiar | rig ludzki CloudRiga | zbudowany czworonog |
|---|---:|---:|
| kosci w szablonie | 66 | 15 |
| kosci w gotowym rigu | 474 | 133 |
| **kosci deformujacych** | **71** | **15** |
| rozjazdow miedzy nazwa `DEF-` a flaga | **0** | **0** |

Kosci deformujace to **dokladnie `DEF-<nazwa kosci szablonu>`, jeden do
jednego**, zero rozjazdow w obie strony. To znaczy, ze **szkielet, ktory
trafi do silnika, jest dokladnie tym szablonem, ktory zaprojektowales**,
tylko z przedrostkiem. Nie trzeba zadnej listy wyjatkow.

**Gotowego szablonu czworonoga CloudRig nie ma** - w pliku z szablonami sa
dwa, oba ludzkie, a slowa `quadruped`, `animal` i `paw` daja w dokumentacji
zero trafien przy dzialajacej kontroli. Ale szablon pisze sie w Pythonie
w kilkunastu linijkach, bo CloudRig jest zestawem klockow:

- kregoslup jako `Spine: IK/FK`
- szyja i ogon jako `Chain: FK`
- **tylna noga czterokostna jako `Limb: Biped Leg`** - ta sama mechanika co
  lapa palcochodna, razem z rolowaniem stopy
- przednia noga jako `Limb: Generic`
- do wtornego ruchu ogona jest osobny `Chain: Physics`, a do pojedynczego
  piora `Feather`

Generacja przez `bpy.ops.pose.cloudrig_generate()` zwrocila `{'FINISHED'}`
**w 0,6 sekundy, w calosci bezokiennie**.

## What didn't work

Cztery rzeczy wywrocily uruchomienie bezokienne, kazda z osobnym objawem:

1. **Wlaczenie dodatku z `default_set=False`.** Preferencje wychodza jako
   `None` i rejestracja sie wysypuje. Musi byc `default_set=True`.
2. **Pominiecie inicjalizacji palety kolorow kosci.** Generacja pada na
   pustym wyliczeniu. Trzeba raz wywolac
   `bpy.ops.preferences.set_bone_color_presets()`.
3. **Brak wlaczenia szablonu.** Bez `obj.cloudrig.enabled = True` operator
   mowi, ze nie widzi szablonu w kontekscie, co wyglada jak blad szukania.
4. **Kosci wewnatrz lancucha bez `use_connect=True`.** Komunikat brzmi
   „Chain must be exactly 3 connected bones" i mysli sie wtedy o dlugosci
   lancucha, a chodzi o polaczenie.

## Pulapka przy eksporcie, ktorej liczba nie zdradza

Eksport FBX z `use_armature_deform_only=True` dal **25 kosci, a nie 15**.
Eksporter **doklada kosci posrednie potrzebne do zachowania hierarchii**.
Odsiew po fladze dziala i zbija 133 do 25, ale **nie zejdziesz do samych
kosci `DEF-`**. Kto zaplanuje budzet kosci na podstawie liczby deformujacych,
pomyli sie o rzad kilkudziesieciu procent.

Dla porownania te same eksporty bez odsiewu: czworonog 133 kosci i 124 KB,
rig ludzki 474 kosci i 453 KB.

## Transferability

To nie jest lekcja o jednym dodatku, tylko o **domyslnych ustawieniach, ktore
przenosza sie do wyniku eksportu**. Regula ogolna: przy kazdym generatorze
rigu zapytaj, **czy flaga deformacji jest wlaczana z decyzji, czy z rozpedu**.
Jesli z rozpedu, to lista rzeczy do usuniecia bedzie rosla sama i cicho sie
rozjedzie z kazda wersja szablonu.

Ta sama mysl dotyczy kazdego potoku, w ktorym narzedzie posrednie ma
domyslne zachowanie „wlacz wszystko": mapy UV, zestawy materialow, kolekcje.
**Taniej jest wybrac narzedzie z domyslna odmowa niz pisac odsiew po fakcie.**

## Related

- [[20260809-1740-rigify-wolf-nie-jest-rigiem-do-gry]] - pomiar, ktory ten problem nazwal
- [[20260805-1815-rigify-ma-gotowe-szkielety-zwierzat]] - dlaczego zaczelismy od Rigify
- [[20260811-0940-cena-u-posrednika-to-nie-cena-u-producenta]] - z tego samego dnia, ta sama rodzina: domyslne zachowanie warstwy posredniej udajace fakt o zrodle

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260805-1815-rigify-ma-gotowe-szkielety-zwierzat|Blender ma w standardzie gotowe szkielety zwierzat (Rigify), zanim siegniesz po AI]] - wspolne: czworonogi, rigging, low-poly
- [[20260809-1740-rigify-wolf-nie-jest-rigiem-do-gry|Metarig wilka w Rigify to rig filmowy, nie growy - do gry idzie basic_quadruped]] - wspolne: czworonogi, low-poly, blender
- [[20260531-2000-blender-mesh-only-fbx-for-mixamo|Batch-extract clean mesh-only FBX from rigged .blend for Mixamo re-rig]] - wspolne: rigging, fbx, blender
- [[20260730-1950-proxy-clothing-tangential-smoothing|Ubrania proxy na low-poly ciele: wygładzanie styczne zamiast laplasjanu]] - wspolne: rigging, low-poly, blender
- [[20260704-2330-blender-unity-flat-panel-dual-face-texture|Blender flat panel textured on one face renders BLANK in Unity (axis-flip picks the wrong face)]] - wspolne: fbx, low-poly, blender
- [[20260802-0210-fbx-blender-unity-obrot-i-tekstury|20260802-0210-fbx-blender-unity-obrot-i-tekstury]] - wspolne: fbx, low-poly, blender
<!-- /POWIAZANE:auto -->
