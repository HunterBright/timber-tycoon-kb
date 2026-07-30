---
type: anti-pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, physics, tunneling, ccd, terrain, spawn, discrete]
date: 2026-07-23
status: draft
---

# Spawn na Y rodzica + jednostronny teren zerowej grubości + Discrete = obiekty pod mapą

## Anti-pattern

Spawnowanie fizycznych obiektów (kłody po ścięciu pnia) na wysokości ORYGINALNEGO obiektu
(baza drzewa) z rozrzutem w poziomie wzdłuż zbocza. Na zboczu w górę obiekt rodzi się POD
powierzchnią low-poly terenu (siatka jednostronna, zerowej grubości, backfaces niewidoczne
dla fizyki - `QueriesHitBackfaces=0`) i spada w nieskończoność. Na zboczu w dół wisi kilka
metrów nad gruntem, rozpędza się i przy `Discrete` tuneluje przez cienką siatkę
(prog: przemieszczenie na krok fizyki > połowa grubości collidera; przy dt=0.02 i kłodzie
~0.3 m to już ~7.5 m/s, czyli spadek z ~3 m). Objaw mylący: "czasami 1 z 3 kłód znika" -
tylko ta, której XZ trafi na niekorzystne zbocze.

## Poprawny wzorzec (trzy warstwy obrony)

1. **Spawn**: raycast gruntu per obiekt pod jego FINALNYM XZ (z góry, maska gruntu,
   `QueryTriggerInteraction.Ignore` bo drogi/strefy mają triggery) → Y = grunt + prześwit.
2. **Prefab**: `ContinuousSpeculative` na Rigidbody - NIE `ContinuousDynamic`, jeśli obiekt
   bywa przełączany na kinematic (optymalizatory, minigry) - kinematic+ContinuousDynamic
   sypie ostrzeżeniami; Speculative jest legalne w obu stanach i łapie cienkie siatki.
3. **Siatka ratunkowa**: próg ratunku TUŻ pod dnem mapy (nie -40 przy dnie -3), i uwaga na
   interakcje z distance-cullingiem fizyki: optymalizator NIE może zamrażać (isKinematic)
   ciał W LOCIE, bo zamrożona kłoda wisi w powietrzu nad progiem ratunku na zawsze.
   Warunek zamrożenia: odległość ORAZ prędkość poniżej progu.

## Diagnoza na przyszłość

"Obiekt czasem wpada pod teren" to prawie nigdy "collider nakłada się za późno".
Sprawdź w tej kolejności: (1) Y spawnu vs realny grunt pod XZ, (2) tryb wykrywania kolizji
i prędkość spadku vs grubość collidera, (3) czy cokolwiek zamraża/teleportuje ciało w locie.
