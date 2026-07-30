---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [unity, physics, meshcollider, convex, raycast, minigame]
severity: high
time_lost: "playtest + ~2h diagnozy"
date: 2026-07-13
status: draft
applies_to: [unity]
---

# Convex MeshCollider na skorupie połyka jej wnętrze - promienie nigdy nie trafią w części w środku

## Problem

Minigra lakierowania mebla: gracz obraca szafę tyłem do kamery, celuje sprayem w tylną ściankę - i **nic się nie dzieje**. Tylna ścianka świeci "niepolakierowana" do końca fazy, z ŻADNEGO kąta nie da się jej trafić. To samo dotyczyło półek regału i wnętrz szafek.

Objawy poboczne, które fałszywie kierowały diagnozę gdzie indziej:
- faza nigdy nie kończyła się automatycznie („pokryto wszystko") - budżet farby wypalał się w pustkę;
- wynik fazy był z góry zablokowany (~70/100), więc najwyższa jakość produktu była **matematycznie nieosiągalna** dla części mebli;
- gracz zgłosił to jako „nie mogę polakierować tylnej ściany, nie wiem czemu" - brzmiało jak bug kamery albo backface cullingu.

## Root cause

Collidery części tworzone w runtime miały `convex = true`:

```csharp
var mc = renderer.gameObject.AddComponent<MeshCollider>();
mc.convex = true;   // <-- BUG
```

Korpus mebla to **skorupa** (dwa boki + dno + góra, przód i tył otwarte). Otoczka wypukła (convex hull) takiej skorupy to **lite pudło wypełniające całą objętość mebla**. Tylna ścianka, półki i drążek są w środku tego pudła.

Raycast wybierający NAJBLIŻSZE trafienie zawsze trafiał najpierw w otoczkę korpusu (indeks 0), a nigdy w część leżącą w jej środku. Część nie była „źle widoczna" - była **fizycznie nieosiągalna**, niezależnie od kąta kamery.

Meble bez korpusu (taboret, stół, ławka) działały bez zarzutu - dlatego bug wyglądał na „problem konkretnego modelu", a nie na systemowy.

## Solution

1. `mc.convex = false` - collider bierze dokładny kształt siatki, więc promień z tyłu wlatuje przez otwarty tył skorupy i trafia w tylną ściankę. `Collider.Raycast` na non-convex MeshColliderze działa bez Rigidbody; PhysX nie re-cookuje przy obracaniu (przebudowa kształtu następuje tylko przy zmianie skali albo siatki).
2. Osobno: część **naprawdę** zamknięta w środku (półka za zamkniętymi drzwiami) pozostaje nieosiągalna także przy dokładnym colliderze - to nie jest bug, tylko geometria. Takie części trzeba **wykryć i wykluczyć z wymagań** (u nas: auto-zaliczone, poświata zgaszona), inaczej faza nie ma jak się domknąć.

Do wykrycia: sonda promieni z okręgu wirtualnych pozycji kamery; część, w którą nie trafia żaden promień z żadnego kąta, jest z definicji poza zasięgiem gracza.

## What didn't work

- Szukanie backface cullingu, `queriesHitBackfaces`, błędu kamery, złych normalnych - wszystkie te hipotezy były fałszywe.
- Obracanie mebla przez gracza (A/D) „powinno pomóc" i nie pomagało - to właśnie ten objaw powinien od razu wskazać na collider, a nie na widoczność: **skoro widać, a nie da się trafić, to problem jest w fizyce, nie w renderze**.

## Transferability

Dotyczy każdej gry, w której gracz celuje promieniem w części złożonego obiektu: malowanie, naprawa, montaż, demontaż, inspekcja, znaczniki „kliknij ten element". Reguła ogólna:

> **Convex collider wolno stosować tylko do brył wypukłych.** Każdy mesh typu skorupa, rama, litera C, U albo pudło z otworem zamienia się w otoczce wypukłej w pełną bryłę i połyka wszystko, co jest w środku.

Druga, szersza lekcja: gdy część systemu jest **nieosiągalna z definicji**, a system wymaga jej „ukończenia", powstaje softlock albo cichy sufit wyniku. Warto mieć automatyczny audyt, który to wykrywa (u nas: test wypisujący tabelę wszystkich 66 części 10 modeli).

## Related
- [[runtime-meshcollider-needs-readable-mesh-in-builds]]
- [[probe-visibility-by-rotating-rays-not-the-object]]
