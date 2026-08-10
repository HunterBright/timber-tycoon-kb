---
title: Camera.Render w trybie edycji rysuje postac ze STARYM skinningiem - dowod z renderu klamie
type: lesson
status: draft
confidence: high
verified: '2026-08-09'
date: 2026-08-09
project: Kerf - Sawmill Tycoon
tags:
- unity
- animacja
- skinning
- render
- dowod
- edit-mode
applies_to:
- Unity 6000.5.1f1
source: _Handoff/drabinka_pracy/diagnoza/raport_diagnozy.txt
severity: high
suggested-category: engine/lessons
time_lost: 'okolo pol dnia (blad rozpoznany dopiero w nastepnej sesji)'
---

# Camera.Render w trybie edycji rysuje postac ze STARYM skinningiem

## Problem

Narzedzie edytorowe renderowalo klatki cyklu animacji, zeby pokazac rezyserowi
kandydatow do wyboru: instancja prefabu postaci, `PlayableGraph` w trybie recznym,
`SetTime` -> `Evaluate` -> `Camera.Render()` -> `ReadPixels` -> PNG.

Na renderach postac stala nieruchomo. W tej samej petli, miedzy `Evaluate` a renderem,
odczyt kosci mowil, ze dlonie wedruja **40 cm w ukladzie swiata**. Powstal falszywy wniosek
"animacja nie dziala / retarget przycina ruch ramion" i pol dnia szukania winy w klipie,
w awatarze i w pomiarze. Wina byla w drodze do obrazu.

## Root cause

Skinning `SkinnedMeshRenderer` nie jest przeliczany przy recznym `Camera.Render()`
w trybie edycji. Silnik robi to w swoim kroku petli klatki, ktory poza trybem gry nie
leci dla obiektu spoza kadru edytora. Kosci maja juz nowa poze, a wierzcholki na obrazie
zostaja w poprzedniej.

Rozstrzygniecie: trzy niezalezne miary tej samej klatki.

| miara | Praca_B (kandydat) | idle (wzorzec) |
|---|---|---|
| kosci (swiat) | dlonie 40,2 cm, biodra 1,4 cm | 0,6 cm |
| siatka `BakeMesh` | najdalszy wierzcholek 66,4 cm | 1,7 cm |
| piksele, render zwykly | 5,1% pikseli | 1,2% |
| piksele, render z pieczonej siatki | **15,1% pikseli** | 1,8% |

Rozjazd miedzy trzecim a czwartym wierszem to caly bug. Siatka deformuje sie poprawnie -
tylko nie trafia na obraz.

## Solution

Przed renderem upiec siatke i narysowac ja jako zwykly `MeshRenderer`:

```csharp
foreach (var s in postac.GetComponentsInChildren<SkinnedMeshRenderer>(true))
{
    var m = new Mesh();
    s.BakeMesh(m, true);                       // true = z uwzglednieniem skali
    var go = new GameObject("Bake_" + s.name);
    go.transform.SetPositionAndRotation(s.transform.position, s.transform.rotation);
    go.AddComponent<MeshFilter>().sharedMesh = m;
    go.AddComponent<MeshRenderer>().sharedMaterials = s.sharedMaterials;
    s.enabled = false;                          // oryginal na czas kadru gasimy
}
// ... Camera.Render() ...
// po kadrze: skasowac tymczasowe obiekty i siatki, wlaczyc oryginaly
```

Do tego druga miara, ktora nie moze sklamac o tym, co widac: **srednia roznica sasiednich
klatek renderu**. Kosci moga jechac, a obraz stac; jesli liczymy tylko na kosciach, nie
mierzymy dowodu, tylko intencje.

## What didn't work

- `animator.cullingMode = AlwaysAnimate` - dotyczy Animatora, nie skinningu siatki.
- `skinnedMeshRenderer.updateWhenOffscreen = true` - **zmierzone: zero roznicy**
  (5,40/765 przed i po, co do drugiego miejsca).
- Porownanie dwoch klatek wybranych na oko - obie wypadly z tej samej fazy cyklu,
  wiec brak roznicy niczego nie dowodzil. Klatki do porownania wybieraj z pomiaru
  (najnizsza i najwyzsza), nie wzrokiem.

## Transferability

Dotyczy kazdego projektu Unity, ktory renderuje postacie z animacji poza trybem gry:
drabinki wariantow do wyboru, generatory ikon, arkusze kontaktowe do przegladu, zrzuty
do dokumentacji, bramki wizualne w CI. Wszedzie tam "render pokazuje X" jest twierdzeniem
o drodze do obrazu, nie o animacji - dopoki nie pieczesz siatki.

Szersza zasada, niezalezna od silnika: **gdy dwa niezalezne pomiary tego samego zjawiska
sie rozjezdzaja, nie wybieraj tego, ktory pasuje do hipotezy - dolóz trzeci, ktory rozstrzyga.**
Tutaj trzecim byl `BakeMesh` i to on wskazal, ze klamie dowod, a nie przedmiot.

## Related

- [[20260809-1145-kimodo-prompt-kropka-dzieli-ruch]] - ta sama rura animacji
- bramka kalibrowana na dzialajacym wzorcu: kandydat uznany za ruch dopiero powyzej
  trzykrotnosci tego, co daje obecna animacja stania
