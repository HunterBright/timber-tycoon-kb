---
title: 'Wydanie gry na Steam: sekwencyjne recenzje + twardy zegar Coming Soon'
type: pattern
status: active
confidence: medium
verified: ''
date: '2026-07-24'
project: Kerf - Sawmill Tycoon
tags:
- steam
- steamworks
- release
- publishing
- coming-soon
- steampipe
- pricing
- timeline
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Wydanie gry na Steam: sekwencyjne recenzje + twardy zegar Coming Soon

## When to use
Przy planowaniu premiery DOWOLNEJ gry na Steam (Steamworks). Szczegolnie gdy ktos zaklada, ze "mamy gotowy build + wypelniona strone = wydajemy w kilka dni". Realny czas to **3-5 tygodni**, bo o terminie decyduja procesy Valve, nie Twoja praca.

## Steps
Kolejnosc jest wymuszona zaleznosciami - nie da sie "zglosic wszystkiego naraz na koniec":
1. **NAJPIERW: umowa podatkowo-bankowa Steam (Payee/Tax interview) + uprawnienia konta** (Manage pricing, Publish). Weryfikacja: dni-tygodnie. Nie widnieje jako checkbox na liscie blokerow, a blokuje publikacje wyceny i sprzedaz.
2. Praca wlasna (rownolegle, 1-3 dni): wgranie builda (SteamPipe/steamcmd), opcje uruchamiania, sync OS, ikony, zwiastun, cena.
3. Publikacja USTAWIEN aplikacji (zakladka Publish) - inne "publish" niz Set Build Live.
4. Cena -> recenzja Valve (1-2 dni robocze) -> osobne klikniecie OPUBLIKUJ wycene.
5. Zgloszenie STRONY sklepu do recenzji (3-5 dni roboczych; zglaszaj 7 dni wczesniej). **Strona idzie PRZED buildem.**
6. Po zatwierdzeniu: "Post as Coming Soon" -> **startuje twardy zegar min. 14 dni** (liczy od kliknniecia, NIE od zatwierdzenia).
7. Zgloszenie BUILDA do OSOBNEJ recenzji Valve (rownolegle w oknie 14 dni; oba depoty na branchu "default").
8. Po >=14 dniach Coming Soon + zatwierdzonym buildzie: reczne "Release App" (gra sama sie nie wyda).

## Why this works
Steam ma **dwie niezalezne recenzje** (strona sklepu ORAZ build) i **jeden nieskracalny zegar** (14 dni publicznego Coming Soon dla nowego produktu). Liczy sie NAJDLUZSZA sciezka, nie suma - dominuje zegar 2 tygodni. Recenzje sa w dni ROBOCZE (weekendy/swieta USA nie licza).

## Trade-offs
Planowanie z buforem "traci" kilka dni, gdy wszystko pojdzie gladko - ale brak buforu = poslizg premiery o cala runde recenzji (kolejne 3-5 dni) albo o zablokowana zmiane daty.

## Variants / GOTCHA transferowalne
- **Build wgrany != build grywalny**: po SteamPipe trzeba osobno "Set Build Live on branch". Bezpiecznie testowac na branchu z haslem przez klient Steam (Properties > Betas) przed premiera.
- **Cena siedzi na PAKIECIE, nie na aplikacji**; wpisanie ceny to propozycja, publikacja wyceny to osobny krok po recenzji Valve.
- **Zniejszka premierowa (Launch Discount, max 40%, 7-14 dni): tylko PRZED premiera**, potem znika bezpowrotnie. Po premierze 30-dniowe okno bez zmian ceny; podwyzka blokuje znizki na 30 dni.
- **Dwa rozne "Publish"**: Set Build Live (pliki gry) vs zakladka Publish (ustawienia: opcje startu, OS, depoty). Trzeba OBA.
- **"Obslugiwane platformy zgodne"** = spojnosc 4 miejsc (Supported OS + OS na depotach + opcje uruchamiania per OS + checkboxy na stronie sklepu) + oba depoty wpiete w pakiet.
- **macOS przez SteamPipe** potrafi zgubic bit wykonywalny w .app / uniewaznic notaryzacje Apple - najlepiej wgrywac build Maca z Maca (builder_osx).
- **Steam Cloud bez kodu (Auto-Cloud)**: dla gry piszacej save na dysk wystarcza Auto-Cloud (folder + pattern), zamiast integracji Cloud API. Celowac w podfolder zapisow, nie caly persistentDataPath (ustawienia per-maszyna i telemetria maja zostac lokalne).
- App Icon aktualnie **184x184 JPG** (nie stare 32x32) - ale przy grafikach ostatecznym zrodlem prawdy jest pole w panelu.
