---
title: Sonda mierzyła JAK zbudować, nie sprawdziła CZY trzeba budować
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-13'
date: 2026-08-13
project: TelegramBridge
tags:
- proces
- spec
- research
- claude-code
- decyzje
applies_to: []
source: 'Mostek: zbudowany bot Telegrama, gdy Claude Code ma wbudowany Remote Control i oficjalny kanał Telegram'
severity: high
suggested-category: process/anti-patterns
time_lost: '~4 h implementacji'
---

# Sonda mierzyła JAK zbudować, nie sprawdziła CZY trzeba budować

## Problem

Powstał kompletny SPEC (9 agentów researchu, panel 3 projektów, 3 sędziów, weryfikacja
adwersaryjna 3 recenzentów, 31 naniesionych uwag) na własnego bota Telegram ↔ Claude Code.
Dokument miał obowiązkową „Fazę 0 — sonda", która **zmierzyła empirycznie** kształty JSON
z CLI, zachowanie `--resume`, deny-listy, kodowanie UTF-8, mechanikę `spawn()` na Windows.
Implementacja przeszła wszystkie testy i działała.

Po wdrożeniu użytkownik zapytał: *„czy to prawda, że Claude Code ma remote session?"*

Ma. I to trzy warianty, z których dwa pokrywały cel projektu:

* **Remote Control** — sterowanie lokalną sesją z aplikacji Claude na telefonie; zdjęcia
  z telefonu, zgody na uprawnienia, natywne push. Flaga `--remote-control` **jest w `claude --help`**.
* **Channels** — oficjalna wtyczka **Telegram**: wiadomość z Telegrama wpada do sesji na
  Twoim komputerze, odpowiedź wraca. Pięć komend instalacji.

Zbudowane rozwiązanie zachowało jedną realną przewagę (działanie non-stop i wstawanie po
restarcie) i zostało zredukowane do ~150 linii dozorcy pilnującego oficjalnej funkcji.
Reszta — ok. 1000 linii — poszła do archiwum.

## Root cause

Łańcuch rozumowania w SPEC-u był **poprawny wewnętrznie, ale zaczynał się o jeden krok za późno**:

> Agent SDK wymaga `ANTHROPIC_API_KEY` → użytkownik ma subskrypcję, nie klucz →
> więc silnikiem musi być headless CLI (`claude -p`) → więc trzeba zbudować most.

Każdy krok prawdziwy. Brakowało kroku zerowego: **„czy narzędzie nie ma już tej funkcji?"**

Wzmacniacze błędu:

1. **Research celował w mechanizm, nie w cel.** Pytania brzmiały „jak SDK się uwierzytelnia",
   „jaki jest format stream-json" — a nie „jak sterować Claude Code z telefonu".
   Research odpowiada na zadane pytanie, nie na to, które należało zadać.
2. **Sonda odziedziczyła założenie.** Faza 0 była wzorowo empiryczna, ale mierzyła wyłącznie
   *jak* rozmawiać z CLI. Ani jedno z 10 pytań sondy nie brzmiało *„czy to w ogóle konieczne"*.
   Rygor metodologiczny **poniżej** błędnego założenia tylko uwiarygodnia wynik.
3. **Część funkcji jest niewidoczna w `--help`.** `--channels` jest w research preview i
   **celowo nie pojawia się w pomocy** (dokumentacja mówi to wprost). Automatyczne
   przeszukanie `--help` jej nie znajdzie. Ale `--remote-control` **tam było** — i zostało
   przegapione, bo nikt nie szukał, skoro decyzja o budowie już zapadła.
4. **Impet dokumentu.** SPEC z fazami, testami akceptacyjnymi i weryfikacją adwersaryjną
   wygląda na przemyślany do końca. Im bardziej dopracowany dokument, tym trudniej
   zakwestionować jego przesłankę — recenzenci sprawdzali *poprawność planu*, nie
   *zasadność projektu*.

## Solution

**Do Fazy 0 każdej sondy dopisać pytanie 0.0, przed wszystkimi technicznymi:**

> „Czy narzędzie/platforma, którą chcę oprogramować, nie ma już tej funkcji wbudowanej —
> także jako beta, research preview albo oficjalna wtyczka?"

Konkretnie dla ekosystemu Claude Code:

```
claude --help                          # pełna pomoc, nie tylko grep pod tezę
```
Plus **przeczytanie dokumentacji**, bo funkcje w research preview bywają ukryte przed `--help`:
* `code.claude.com/docs/en/remote-control` — sterowanie lokalną sesją z telefonu
* `code.claude.com/docs/en/channels` — Telegram / Discord / iMessage do sesji
* `code.claude.com/docs/en/claude-code-on-the-web` — sesje w chmurze
* `code.claude.com/docs/en/scheduled-tasks`, `/desktop#sessions-from-dispatch`
* `code.claude.com/docs/llms.txt` — **indeks całej dokumentacji**, dobre miejsce na start

Dokumentacja Remote Control ma tabelę **„Choose the right approach"** porównującą
Dispatch / Remote Control / Channels / Slack / self-hosted / scheduled tasks. Jedna tabela,
która rozstrzygnęłaby cały projekt w 5 minut.

**Gdy jednak budujesz mimo istniejącej funkcji** — nazwij wprost, czego wbudowane nie robi.
Tutaj poprawna odpowiedź brzmiała: *„Remote Control umiera z terminalem i nie wstaje po
restarcie"*. To jest **jedna** brakująca właściwość, więc buduje się **tylko ją** (dozorca,
~150 linii), a nie całą warstwę od zera (~1000 linii).

## What didn't work

- **Weryfikacja adwersaryjna 3 recenzentów.** Wyłapali 31 uwag technicznych — i ani jednej
  dotyczącej sensu przedsięwzięcia. Recenzenci dostali pytanie „czy ten plan jest dobry",
  więc odpowiedzieli na nie. **Recenzja planu nie jest recenzją decyzji.** Żeby wyszła
  przesłanka, trzeba osobno zapytać: „co sprawiłoby, że ten projekt jest zbędny?".
- **Skala researchu.** 9 agentów badających SDK, API Telegrama i audyt maszyny. Więcej
  researchu w złym kierunku nie zawraca go — tylko podnosi pewność siebie.
- **Rygor sondy.** Faza 0 była dokładnie tym, czym powinna: pomiarem zamiast założeń.
  Ale mierzyła po stronie już podjętej decyzji.

## Transferability

Dotyczy każdego projektu, w którym pisze się „warstwę" wokół cudzego narzędzia: wtyczki
do Unity, skrypty do Blendera, integracje z Discordem, automaty do Steama, obudowy na CLI.
Ryzyko rośnie, gdy narzędzie rozwija się szybko (Claude Code, Unity 6, Blender 4.x) —
funkcja, której nie było trzy miesiące temu, może istnieć dzisiaj, a wiedza modelu
i pamięć zespołu są z definicji przestarzałe.

**Reguła:** zanim napiszesz pierwszą linijkę obudowy, przeczytaj indeks dokumentacji
narzędzia i poszukaj swojego przypadku użycia. Nie `--help` pod tezę — **spis treści dokumentacji**.
Koszt: 10 minut. Cena pominięcia: cały projekt.

**Druga reguła:** gdy wbudowana funkcja pokrywa 90% celu, buduj **tylko brakujące 10%**.
Kuszące jest przepisać całość „po swojemu, bo i tak już wiem jak" — to jest ta sama pomyłka,
tylko drugi raz.

## Related
- [[20260813-1425-ps1-bez-bom-czytany-jako-ansi]] — druga lekcja z tego samego projektu
- [[20260805-1520-przedawnienie-wiedzy-jest-funkcja-typu-wpisu]] — wiedza o szybko
  zmieniających się narzędziach przedawnia się najszybciej
- [[20260809-1440-bramka-nie-moze-przyznawac-tego-co-sprawdza]] — pokrewne: weryfikacja,
  która nie może wykryć własnego błędu założenia
