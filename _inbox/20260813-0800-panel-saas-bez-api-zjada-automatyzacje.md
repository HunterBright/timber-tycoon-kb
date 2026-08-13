---
title: Panel SaaS bez API zjada automatyzację przeglądarkową — rozpoznaj to po trzech objawach
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-13 — trzy nieudane podejścia do panelu carl.gg'
date: 2026-08-13
project: Discord_Studio (MGDB Studio)
tags:
- automatyzacja
- przegladarka
- saas
- discord
applies_to: []
source: ''
suggested-category: workflow/anti-patterns
---

# Panel SaaS bez API zjada automatyzację przeglądarkową

## Co próbowaliśmy
Wyklikać agentem konfigurację bota Discorda w panelu `carl.gg`: trzy wiadomości z rolami
do odbioru, w tym jedna z dziesięcioma parami emoji→rola. Dwie prostsze grupy (dwie i cztery
pary) poszły. Grupa z dziesięcioma parami **nie przeszła w trzech podejściach** i za każdym
razem cała robota przepadała.

## Dlaczego nie działa
Trzy cechy panelu, każda z osobna do przeżycia, razem zabójcze:

1. **Lista wielokrotnego wyboru, która nie zamyka się po wyborze.** Kolejne kliknięcie —
   w cokolwiek — trafia w nią i dokłada losową pozycję. Tak do roli „• Green" doszła „• Pink",
   a wykryć to można wyłącznie zrzutem ekranu po każdym kroku.
2. **Escape zamyka cały formularz, nie samą listę.** Naturalne „zamknij dropdown" kasuje
   dwadzieścia minut pracy bez pytania i bez ostrzeżenia.
3. **Stan pól przeżywa między iteracjami** — pole wyszukiwania emoji nie czyści się, więc
   drugie zapytanie brzmi „red circleorange circle" i nic nie znajduje.

Do tego dochodzi zmienna skala renderowania strony (te same elementy raz na y=548, raz na
y=668) i sypiące się zrzuty ekranu przy dłuższych formularzach — czyli współrzędne trzeba
odczytywać na nowo po **każdym** kroku, co znosi cały zysk z automatyzacji.

## Jak to rozpoznać wcześnie
Po pierwszej iteracji policz **kroki na jednostkę pracy**. Jeśli jedna para (dwa pola)
kosztuje trzy wywołania narzędzia i zrzut ekranu, to dziesięć par kosztuje trzydzieści —
a przy pierwszej pomyłce zaczynasz od zera, bo formularz jest transakcyjny (zapis dopiero
na końcu). **Formularz transakcyjny z liczbą powtórzeń > 5 to sygnał, żeby oddać go
człowiekowi**, zanim się zacznie.

## Co robić zamiast
- **Podziel po granicy zapisu.** Trzy osobne wiadomości po 2–4 pary przeszły bez problemu;
  jedna wiadomość z dziesięcioma parami nie przeszła nigdy. Jeśli można rozbić na mniejsze
  zapisy, rób to nawet kosztem brzydszego wyniku.
- **Oddaj powtarzalną część człowiekowi z gotowym przepisem.** Ręka trafia w pole bez
  zrzutu ekranu; dziesięć par to dla człowieka pięć minut, dla agenta pół godziny i loteria.
- **Zawsze sprawdzaj wynik u źródła, nie w panelu.** Tabela w carl.gg pokazywała jeden wpis,
  gdy na serwerze były już dwa — dowodem był odczyt wiadomości i reakcji przez API Discorda.
  Patrz [[20260813-0640-success-z-mostu-mcp-nie-dowodzi-zapisu]].

## Related
- [[20260813-0640-success-z-mostu-mcp-nie-dowodzi-zapisu]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-1745-discord-bot-viewchannel-samoodciecie|Bot Discorda bez Administratora nie ukryje kanału przed @everyone]] - wspolne: discord, automatyzacja
<!-- /POWIAZANE:auto -->
