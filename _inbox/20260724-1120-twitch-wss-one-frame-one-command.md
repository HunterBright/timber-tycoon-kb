---
title: 'Twitch IRC po WebSocket: jedna paczka = jedna komenda (i jak cichy klient to ukryl)'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-24'
project: Kerf - Sawmill Tycoon
tags:
- twitch
- irc
- websocket
- chat
- integration
- silent-failure
- false-positive
applies_to: []
source: ''
severity: critical
suggested-category: integrations/lessons
---

# Twitch IRC po WebSocket: jedna paczka = jedna komenda (i jak cichy klient to ukryl)

## Objaw

Integracja czatu Twitch (anonimowy odczyt, justinfan) pokazuje status "Polaczono", ale zaden
czat nie plynie. Licznik odebranych linii zatrzymany na 2. Ta sama logika na zapasowym
transporcie TCP:6697 dziala.

## Przyczyna nr 1 - protokol

Koncowka `wss://irc-ws.chat.twitch.tv:443` traktuje JEDNA paczke (message) WebSocket jako
JEDNA komende IRC. Wyslanie handshake'u sklejonego w jedna paczke:
`CAP REQ ...\r\nNICK justinfanX\r\nJOIN #kanal\r\n`
konczy sie odpowiedzia `:tmi.twitch.tv CAP * NAK :NICK justinfanX JOIN #kanal` - serwer
wzial NICK i JOIN za smieciowe nazwy capability w ogonie CAP REQ. Rejestracja nigdy nie
zachodzi, serwer nie wysyla powitania 001 ani czatu. Na surowym TCP ten sam sklejony blob
dziala (strumien bajtow jest ciety po \r\n) - stad zdradliwa asymetria transportow.

**Regula: na wss Twitcha KAZDA komenda IRC leci OSOBNA paczka WebSocket** (CAP REQ, NICK,
JOIN, PONG - wszystkie). Wtedy przychodzi pelne powitanie 001-376 + JOIN echo + ROOMSTATE.

## Przyczyna nr 2 - falszywy pozytyw statusu

Detektor "dolaczylem" oparty o `line.Contains(" JOIN #")` lapal... linie ODMOWY CAP NAK,
bo ta cytuje wyslane smieci (`...NAK :NICK justinfanX JOIN #kanal`). Status klamal
"Polaczono" przy calkowicie martwym polaczeniu. Dowodem dolaczenia moga byc TYLKO linie,
ktorych tresci nie kontroluje nadawca: powitanie `001` albo `ROOMSTATE` (i tylko sprawdzane
PRZED dolaczeniem - po dolaczeniu widz moze napisac cokolwiek).

## Przyczyna nr 3 - niema sciezka (to ona ukryla bug na tygodnie)

Klient mial: pusty `catch (Exception) {}` z cichym retry + ZERO logow po "lacze..."
(ani dolaczenia, ani odebranych linii, ani bledow). Smoke test transportu mial kryterium
PASS = join (falszywie pozytywne), wiec swiecil zielono na zepsutym handshake'u, a pole
`welcome=False` w raporcie nikogo nie zaniepokoilo.

Zasady:
- kazdy etap zywej integracji LOGUJE (polaczono / dolaczono / pierwsza wiadomosc / komenda
  rozpoznana / efekt), bledy sieci logowane (z watku tla przez kolejke do glownego, jesli
  logger jest main-thread-only);
- log wyzwalany trescia od obcych (spam komendy) MUSI miec dlawik czasowy;
- kryterium PASS smoke testu = NAJMOCNIEJSZY dowod (welcome ORAZ join), nie najslabszy;
- testy offline (parser, kolejka) NIE zastepuja jednego biegu na zywo z asercja na logu.

## Bonus - dwa gotcha z tej samej naprawy

- Bufor znakow dla statefull `Decoder.GetChars` musi miec `GetMaxCharCount(bajty)`, nie tyle
  co bufor bajtow: znak 4-bajtowy rozciety na granicy paczek dopina sie do PARY zastepczej
  (2 znaki z 1 bajtu) i pelna paczka daje bajty+1 znakow -> ArgumentException.
- Unity UI: obiekt parentowany do canvasu MUSI dostac RectTransform (rozciagniety na canvas),
  inaczej kotwice dzieci licza sie od SRODKA ekranu, nie od rogu.
