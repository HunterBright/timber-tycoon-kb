---
title: Podmiana prymitywu Unity na model FBX po cichu zabija interakcję
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-13'
project: Kerf - Sawmill Tycoon
tags:
- unity
- colliders
- interaction
- raycast
- fbx-import
- smoke-tests
- regression
applies_to: []
source: ''
severity: high
suggested-category: engine/lessons
---

# Podmiana prymitywu Unity na model FBX po cichu zabija interakcję

## Objaw

Obiekt w scenie przestał reagować na klawisz interakcji (E). Zero błędów w konsoli, zero
ostrzeżeń. Logika obiektu działała w 100% poprawnie (testy przechodziły), ale gracz fizycznie
nie mógł go kliknąć. Objaw wyglądał na "bug logiki", a był to bug fizyki.

## Przyczyna źródłowa

Wykrywanie obiektów do interakcji szło przez `Physics.Raycast` po warstwie `Interactable`.
Promień trafia **wyłącznie w collidery**. Obiekt collidera nie miał, bo:

1. Wcześniej podest był `GameObject.CreatePrimitive(PrimitiveType.Cube)` — **prymitywy Unity
   dostają collider z automatu**, więc nikt nigdy nie musiał go dodać jawnie.
2. Ktoś podmienił prymityw na prawdziwy model FBX. **Importer FBX ma `addColliders: 0`
   domyślnie** — model nie wnosi collidera.
3. Kod, który stawiał wizual na obiekcie, dodatkowo **kasował wszystkie collidery** z wizualu
   (żeby nie blokowały raycastów) — więc nie było już żadnej deski ratunku.

Efekt: warstwa `Interactable` była ustawiona poprawnie, komponent interakcji był na obiekcie,
kod był zdrowy — ale nie było **w co trafić promieniem**.

## Dlaczego testy tego nie złapały

Smoke test wołał metodę biznesową **wprost z kodu**:

```csharp
showroom.PlaceFromStand(pedestal);   // PASS - logika zdrowa
```

Test nigdy nie przeszedł przez raycast gracza, więc nie miał szans zauważyć, że obiekt jest
dla gracza niewidzialny. **Test przechodził, gra była zepsuta.**

## Reguła

Gdy podmieniasz prymityw Unity (`CreatePrimitive`) na zaimportowany model, **zawsze dodaj
collider jawnie** — prymityw niósł go za darmo, model nie niesie.

Szerzej: jeśli feature ma **fizyczną powierzchnię interakcji** (raycast, trigger, klik w świecie),
to sam test logiki nie wystarcza. Potrzebny jest test, który sprawdza **klikalność**:

```csharp
// Asercja, która złapałaby ten bug od razu:
foreach (var target in interactables)
{
    Assert(target.GetComponent<Collider>() != null, "brak collidera - E nic nie zrobi");
    Assert(!target.GetComponent<Collider>().isTrigger, "trigger - promień go pominie");
    Assert(target.gameObject.layer == interactableLayer, "zła warstwa");
}
```

## Sygnał ostrzegawczy do zapamiętania

Gdy użytkownik mówi **"X nie działa"**, a kod X jest ewidentnie poprawny i pokryty zielonym
testem — nie szukaj dalej w logice. Sprawdź, czy gracz w ogóle **może dosięgnąć** X:
collider, warstwa, `isTrigger`, zasięg, czy coś nie zasłania.

## Klasa błędu (nazwa własna)

**"Logika zdrowa, klikalność martwa"** — cała rodzina bugów, w których system działa idealnie,
ale nie ma jak go uruchomić z poziomu gracza. Testy jednostkowe/integracyjne są na nią ślepe
z definicji, bo omijają warstwę wejścia.
