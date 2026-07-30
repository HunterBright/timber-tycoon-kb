---
title: 'Dopasowanie SFX o stałej długości do akcji o zmiennej długości: pętla + wygaszenie'
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-06-25'
project: Kerf - Sawmill Tycoon
tags:
- unity
- audio
- sfx
- looping
- fade
- minigame
- pooling
- soundbank
applies_to: []
source: ''
suggested-category: engine/patterns
---

# Dopasowanie SFX o stałej długości do akcji o zmiennej długości: pętla + wygaszenie

## Problem
Akcja w grze trwa zmienną i często KRÓTSZĄ ilość czasu niż plik dźwiękowy
(praca maszyny, jazda, kopanie, obrywanie gałęzi, upadek drzewa zatrzymany przeszkodą).
Odtworzenie pełnego klipu „ciągnie się" po akcji; twarde ucięcie daje słyszalny „klik".

## Wzorzec
Trzy strategie, dobierane wg typu dźwięku:

1. **PĘTLA + wygaszenie (StopWithFade)** — dla akcji ciągłych o nieznanej długości.
   Klip gra zapętlony od startu akcji; na końcu krótkie wygaszenie (~0.1–0.15 s) zamiast Stop().
   Długość pliku przestaje mieć znaczenie — pętla docina się sama do akcji.

   Minimalne API w centralnym AudioManagerze:
   ```csharp
   AudioSource PlayLoop(string id, Vector3? pos = null, float pitch = 1f); // zwraca uchwyt
   void StopWithFade(AudioSource src, float fade = 0.12f);                  // fade → Stop → reset → zwrot do puli
   ```
   Wywołujący trzyma uchwyt, opcjonalnie moduluje pitch w trakcie, woła StopWithFade na KAŻDYM
   wyjściu akcji (sukces / anulowanie / błąd). Uchwyt = źródło z puli; po fade wraca do puli.

2. **JEDNORAZOWY + lekka wariacja pitcha** — dla dyskretnych ciosów/uderzeń (cios siekiery, klik).
   Krótki klip per zdarzenie, `pitch = 1 + Random(-0.06, 0.06)` żeby nie powtarzał się monotonnie.
   Naturalnie dopasowany (jedno zdarzenie = jeden dźwięk); kolejne zdarzenia nakładają się przez pulę.

3. **PRZEJŚCIE sterowane stanem** — gdy akcja kończy się nieprzewidywalnie (np. upadek drzewa
   zatrzymany kolizją). Pętla świstu z pitchem rosnącym z prędkości, zatrzymywana DOKŁADNIE
   w momencie wykrycia końca (kolizja/kąt), zaraz po niej jednorazowy dźwięk uderzenia.
   Klucz: pętlą steruje stan fizyki/gry, nie z góry założony czas.

## Higiena
- Pętle żyjące długo (silnik, maszyna) zajmują źródło z puli na cały czas akcji → zwiększ pulę SFX
  o zapas (np. 10→16) albo daj im dedykowane źródła.
- Głośność trzymaj centralnie (pole `volume` per wpis w banku dźwięków), nie rozsianą po call-site'ach —
  pozwala wypoziomować całość bez grzebania w kodzie.
- Pętle powtarzalne ustawiaj ciszej niż pojedyncze uderzenia (mniej męczą przy długim słuchaniu).
- Wspólny gating: dźwięki sterowane ruchem/stanem gracza pytaj o ten sam „czy zajęty", który blokuje
  sterowanie (jedno źródło prawdy) — patrz lekcja o zamrożonej CharacterController.velocity.
