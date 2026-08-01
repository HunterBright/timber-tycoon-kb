---
title: Balansowanie ekonomii progresu metodą „dwóch zegarów" + koperty przychodu
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-04'
project: Kerf - Sawmill Tycoon
tags:
- economy
- balancing
- progression
- tycoon
- simulation
- pacing
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Balansowanie ekonomii progresu metodą „dwóch zegarów" + koperty przychodu

## When to use
Gdy trzeba wycenić ceny ulepszeń / koszty / nagrody w grze z progresją poziomową (tycoon, RPG, idle),
a wartości są placeholderami. Szczególnie gdy postęp gracza napędzają DWA różne zasoby naraz
(np. reputacja/XP zdobywane za LICZBĘ akcji, oraz pieniądze zdobywane za WARTOŚĆ akcji).

## Steps
1. **Zidentyfikuj zegary progresu.** Rozdziel to, co gatuje awans (np. reputacja = liczba zamówień)
   od tego, co gatuje zakupy (np. pieniądze = wartość zamówień). Często jeden już ma wbudowany kształt
   „wolniej z każdym poziomem" - nie dubluj go na ślepo.
2. **Policz KOPERTĘ przychodu, nie jedną liczbę.** Dla każdego poziomu wylicz trzy scenariusze:
   FLOOR (gracz zawsze dostaje najsłabsze wyniki), CEIL (zawsze najlepsze), AVG (środek).
   Zsumuj narastająco do końca gry. To daje realny budżet, do którego dopasowujesz wydatki.
3. **Wyraź tempo w naturalnej jednostce, nie w walucie.** „Ile AKCJI kosztuje awans" (koszt / przychód-na-akcję)
   czyta się lepiej niż surowe kwoty i od razu widać, czy pieniądze nie stają się ścianą względem drugiego zegara.
4. **Dopasuj wydatki jako rosnący % przychodu.** Wczesne poziomy: mały % (nadwyżka → „fajnie, kupujesz swobodnie").
   Późne: większy % (trzeba odkładać → „wolniej"). Kręgosłup obowiązkowy trzymaj poniżej FLOOR (zawsze osiągalny);
   luksusy niech konsumują nadwyżkę FLOOR→CEIL.
5. **Zweryfikuj skryptem symulacyjnym** (np. Python): balans narastający ≥ 0 na każdym poziomie na ścieżce AVG;
   komplet ≤ FLOOR jeśli design zakłada „kup wszystko"; krzywa „akcji na awans" rośnie do końca.
6. **Oddaj decyzje reżyserskie właścicielowi.** Trudność (jak ciasno), endgame (komplet czy luksusy poza zasięgiem),
   długość (ile akcji do maksa) - to wybory feel, nie matematyka. Zapytaj PRZED wpisaniem liczb.

## Why this works
Przychód w takich grach jest zwykle funkcją losową w widełkach - pojedyncza „średnia" ukrywa wariancję.
Koperta FLOOR/AVG/CEIL pokazuje, czy pechowy gracz też przejdzie i ile luzu ma szczęściarz.
Rozdzielenie zegarów ujawnia, który zasób realnie gatuje tempo - często okazuje się, że jeden zegar
już daje pożądany kształt, więc drugi wystarczy dopasować, zamiast przeprojektowywać oba.

## Trade-offs
- Wymaga skryptu symulacyjnego i danych źródłowych (grep progów/cen), nie „na oko" - więcej pracy wstępnej.
- Model zakłada, że przychód/akcja jest w przybliżeniu niezależny od produktu; jeśli gracz realnie kontroluje
  wartość (nie losowa), koperta FLOOR/CEIL trzeba liczyć inaczej (wybór optymalny, nie skrajności losu).

## Variants
- **Jeden zegar:** jeśli progres napędza tylko waluta, pomiń krok 1 - zostaje koperta + rosnący %.
- **„Komplet na maksie" vs „luksusy poza zasięgiem":** dwie różne kalibracje kroku 4 (suma wszystkiego ≤ AVG,
  albo kręgosłup ≤ FLOOR a top-tier tylko na CEIL / po maksie).
- **Case Timber Tycoon (2026-07-04):** reputacja +5/zamówienie (382 do L13), pieniądze z widełek 20-295;
  stary kręgosłup 249k = 3,3× przychodu (76k) → przecena do 37,6k. Sim: scratchpad/progress_sim2.py.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260712-0010-per-sku-price-decay-rotation-exploit|Per-SKU price decay przegrywa z rotacja 2 SKU (gdy odnowa wyprzedza tempo produkcji)]] - wspolne: balancing, simulation, economy
- [[20260714-2215-order-value-topdown-makes-prices-meaningless|Zamówienie losowane OD KWOTY w dół - cennik przestaje cokolwiek znaczyć]] - wspolne: progression, economy, tycoon
- [[worker-simulate-work-cycle|Worker Simulate Work Cycle (No NavMesh/AI)]] - wspolne: simulation, tycoon
- [[carry-capacity-progression-sprint|Carry Capacity Progression + Sprint Advantage]] - wspolne: progression, tycoon
- [[customer-tier-system|Customer Tier System (Regular / Contractor / VIP)]] - wspolne: progression, tycoon
- [[worker-output-quality-distribution|Worker Output Quality Distribution]] - wspolne: progression, tycoon
<!-- /POWIAZANE:auto -->
