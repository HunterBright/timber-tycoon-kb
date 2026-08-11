---
title: Metarig wilka w Rigify to rig filmowy, nie growy - do gry idzie basic_quadruped
type: lesson
status: needs-reproduction
confidence: low
verified: '2026-08-09'
date: 2026-08-09
project: GameDevOS
tags:
- rigify
- blender
- rigowanie
- czworonogi
- unity
- low-poly
applies_to:
- Blender 5.2 LTS
- Unity 6000.5
source: 'pomiar wlasny, blender --background, 2026-08-09'
severity: high
suggested-category: engine/lessons
time_lost: 'pieć dni wiszacego zadania w radarze'
---

# Metarig wilka w Rigify to rig filmowy, nie growy

## Problem

Od 05.08.2026 w radarze wisialo zadanie „jeden potwor przez Rigify, od poczatku
do konca", z pytaniem sformulowanym jako **„ile minut zajmuje dopasowanie kosci
metarigu wilka"**. Pytanie bylo zle postawione, bo zakladalo, ze metarig wilka
jest wlasciwym punktem startu dla potwora low poly.

Pomiar bezokienny (`blender --background`, Blender 5.2.0 LTS) pokazal:

| metarig | kosci metarigu | kosci po wygenerowaniu | kosci deformujace | czas generowania |
|---|---:|---:|---:|---:|
| **wilk** | **190** | **823** | **197** | 4,4 s |
| kot | 174 | 760 | 181 | 4,9 s |
| kon | 70 | 454 | 80 | 0,17 s |
| ptak | 75 | 357 | 79 | 0,15 s |
| rekin | 35 | 157 | 32 | 0,06 s |
| **czworonog podstawowy** | **34** | **283** | **46** | 0,11 s |

Metarig wilka zawiera pelny rig twarzy odziedziczony po metarigu ludzkim:
`nose.001-004`, `lip.T/B`, `brow.T/B` (12 kosci), `lid.T/B` (12), `tongue`,
`teeth.T/B`, `cheek`, `jaw`, oraz - co rozstrzyga sprawe - **`breast.L/R`
i lancuchy palcow `f_pinky`, `f_index`, `f_middle`, `f_ring`** (przednie lapy)
razem z `r_pinky` i reszta (tylne). Do tego `spine` az do `spine.011`.

## Root cause

Metarigi zwierzat w Rigify powstaly przez rozbudowe metarigu ludzkiego, a nie
jako osobna, oszczedna konstrukcja. Sa przeznaczone do animacji filmowej, gdzie
mimika i palce maja znaczenie. **Nikt ich nie odchudzil pod gre**, a nazwa
„wilk" sugeruje gotowy rig zwierzecy, wiec nie sprawdza sie zawartosci.

Dodatkowo eksport FBX **wypuszcza wszystkie 823 kosci**, nie tylko 197
deformujacych, jesli nie odsieje sie kosci sterujacych przed eksportem.

## Solution

**Punktem startu jest `basic_quadruped` (34 kosci), nie `wolf`.** Powody:

1. Ma czyste, growe nazwy: `spine.001-011`, `shoulder.L/R`,
   `front_thigh/front_shin/front_foot/front_toe`, `thigh/shin/foot/toe`,
   `pelvis.L/R`. Zero kosci twarzy.
2. **Recznie trzeba ustawic 23 kosci** przy wlaczonym X-Mirror (12 w osi
   symetrii + 11 z jednej strony), zamiast **132 przy wilku** (74 + 58).
3. Liczba kosci trafia w to, czego naprawde uzywaja modele czworonogow:
   mediana **30 kosci** w probie 60 czworonogow z Mobjaverse.

**Czego `basic_quadruped` nie ma i trzeba dolozyc:** ogona, szyi i glowy.
Lancuch `spine.005-011` pelni role szyi, ale bez osobnej glowy i zuchwy.
Z pomiaru na Mobjaverse: ogon **3-4 kosci**, lancuch nogi **4 kosci** (mediana),
lancuch osiowy **4 kosci** (mediana).

**Kanoniczny budzet rigu potwora, wyprowadzony z pomiaru 53 czworonogow:**

```
  1  korzen (na podlozu, bez ruchu)
  1  miednica
  3  kregoslup
  2  szyja
  1  glowa
  1  zuchwa
  8  noga przednia L+P po 4 kosci
  8  noga tylna L+P po 4 kosci
  3  ogon
 ---
 28  RAZEM   (mediana w zbiorze odniesienia: 30-31 kosci)
```

**Przed eksportem do Unity odsiac kosci sterujace** - inaczej idzie 823 zamiast
197. Kosci deformujace poznaje sie po fladze `use_deform`.

## What didn't work

- **Zalozenie, ze „metarig wilka" znaczy „rig wilka do gry".** Nie znaczy.
- **`bpy.ops.wm.read_factory_settings(use_empty=True)` miedzy pomiarami** -
  wylacza dodatek Rigify, przez co operatory metarigow znikaja i skrypt sypie
  `AttributeError: operator could not be found`. Scene trzeba czyscic recznie
  przez `bpy.data.objects.remove()`.
- **`addon_utils.enable("rigify")`** - rejestruje modul, ale **nie tworzy wpisu
  w preferencjach**, przez co generowanie rigu wywala sie pozniej na
  `RigifyParameters object has no attribute make_custom_pivot`. Dziala dopiero
  `bpy.ops.preferences.addon_enable(module="rigify")`.

## Transferability

Dotyczy kazdego projektu w Unity albo Unreal, ktory potrzebuje zwierzat lub
potworow i rozwaza Rigify jako darmowa alternatywe dla platnych auto-riggerow.
Rozroznienie „rig filmowy kontra rig growy" i pomiar liczby kosci deformujacych
przed wyborem metarigu jest niezalezne od gatunku gry i od silnika.

Osobno transferowalna jest metoda: **zamiast pytac „ile minut zajmie dopasowanie",
policz kosci, ktore czlowiek musi ruszyc** - to jest liczba, ktora da sie zmierzyc
bezokiennie, zanim ktokolwiek otworzy Blendera.

## Related

- [[20260809-1755-mobjaverse-jako-punkt-odniesienia-dla-rigow]] - skad wzielismy mediane 30 kosci
- Blender ma szesc metarigow zwierzat w standardzie: wilk, kot, kon, ptak,
  rekin, czworonog podstawowy (zmierzone `dir(bpy.ops.object)`)

> **Adnotacja weryfikatora 2026-08-10.** Wniosek glowny (startowac z `basic_quadruped`,
> nie z `wolf`) potwierdzony w kodzie zrodlowym Rigify - liczby kosci metarigow zgadzaja
> sie co do jednej (wilk 190, kot 174, kon 70, ptak 75, rekin 35, czworonog 34), podobnie
> jak obecnosc w wilku kosci twarzy, `breast.L/R` i lancuchow palcow. Dwa zdania sa jednak
> **sprzeczne ze zrodlem**. Pierwsze: "czego `basic_quadruped` nie ma i trzeba dolozyc:
> ogona, szyi i glowy" - w pliku `metarigs/Basic/basic_quadruped.py` kosc `spine.003` ma
> przypisany typ `spines.basic_tail`, a `spine.009` typ `spines.super_head`, czyli ogon
> i glowa sa w metarigu skonfigurowane; podrecznik Blendera opisuje ten metarig jako
> zawierajacy "the new tail option in the spine". Brakuje realnie tylko zuchwy. Drugie:
> "132 kosci do ustawienia recznie przy wilku (74 + 58)" - w zrodle wilk ma 26 kosci
> w osi symetrii i po 82 na strone, czyli przy X-Mirror wychodzi 108, a nie 132; skladniki
> 74 i 58 nie odpowiadaja niczemu w pliku. Liczba 23 dla `basic_quadruped` jest poprawna
> (12 osiowych + 11 z jednej strony), ale zawiera `breast.L`, czyli te sama pozostalosc
> po metarigu ludzkim, ktora wpis podaje jako argument rozstrzygajacy przeciw wilkowi.
> Zrodlo: `scripts/addons_core/rigify/metarigs/` w instalacji Blendera 5.2 LTS oraz
> https://projects.blender.org/blender/blender-manual (metarigs).

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260805-1815-rigify-ma-gotowe-szkielety-zwierzat|Blender ma w standardzie gotowe szkielety zwierzat (Rigify), zanim siegniesz po AI]] - wspolne: rigify, czworonogi, low-poly
- [[20260809-1755-mobjaverse-jako-punkt-odniesienia-dla-rigow|Mobjaverse - nazwy kosci sa puste w 80 procentach, wiec to punkt odniesienia dla struktury, nie zrodlo animacji]] - wspolne: rigowanie, czworonogi
- [[20260811-1520-cloudrig-daje-czysty-rig-do-gry-bo-odwraca-domyslne-use-deform|CloudRig daje czysty rig do gry, bo odwraca domyslne use_deform Blendera]] - wspolne: czworonogi, low-poly, blender
- [[20260801-1130-quadriflow-kasuje-uv-i-wagi-bez-jednej-flagi|QuadriFlow kasuje UV i wagi szkieletu, dopoki nie wlaczysz jednej flagi]] - wspolne: low-poly, blender
- [[20260731-1050-rowne-krawedzie-ubran-bisect-plane|20260731-1050-rowne-krawedzie-ubran-bisect-plane]] - wspolne: low-poly, blender
- [[20260801-0826-trellis-2-generator-3d-bez-blokady-ue|TRELLIS.2 jako generator 3D bez blokady licencyjnej w UE]] - wspolne: low-poly, blender
<!-- /POWIAZANE:auto -->
