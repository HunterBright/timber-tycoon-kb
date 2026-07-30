---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: engine/patterns
tags: [twitch, chat, irc, websocket, streaming, networking, threading, unity, mono]
date: 2026-07-23
status: draft
---

# Anonimowy odczyt czatu Twitch w Unity (zero kont, kluczy i kosztow)

## Problem
Gra chce reagowac na czat Twitch streamera (widzowie jako NPC, glosowania, komendy),
ale bez zmuszania streamera do zakladania aplikacji dev, OAuth i utrzymywania serwera.

## Wzorzec (zweryfikowany NA ZYWO 2026-07-23, w buildzie Mono Unity 6)
1. **Transport**: `System.Net.WebSockets.ClientWebSocket` na `wss://irc-ws.chat.twitch.tv:443`.
   Dziala w buildzie Mono + .NET Standard 2.1 bez zadnych pakietow. Zapas: `TcpClient`+`SslStream`
   na `irc.chat.twitch.tv:6697` (tez zweryfikowany).
2. **Handshake anonimowy** (bez PASS!):
   `CAP REQ :twitch.tv/tags twitch.tv/commands` + `NICK justinfan<losowe cyfry>` + `JOIN #kanal`.
   Tagi daja display-name, id wiadomosci, zakresy emotek, CLEARCHAT/CLEARMSG (moderacja).
   Polaczenie jest z konstrukcji read-only - proba wyslania PRIVMSG jest PO CICHU odrzucana.
3. **Watki**: odbior w tle (`Task.Run` + `CancellationTokenSource`), surowe linie do
   `ConcurrentQueue<string>`, oprozniane w `Update()` na glownym watku. PONG na PING odsylac
   OD RAZU z petli w tle (nie czekac na klatke gry). Teardown: Cancel w OnDestroy +
   OnApplicationQuit - inaczej Edytor moze wisiec przy wyjsciu z Play Mode.
4. **Dekodowanie**: `Encoding.UTF8.GetDecoder()` (stanowy), NIE `GetString` per pakiet -
   znak wielobajtowy potrafi byc przeciety na granicy pakietow.
5. **RECONNECT**: rozpoznawac po DOKLADNYM prefiksie `:tmi.twitch.tv RECONNECT`,
   nie po Contains - widz moze napisac slowo RECONNECT w wiadomosci.

## Gotchas
- `UnityEngine.Random` (i cale Unity API) NIE dziala na watku w tle - `System.Random`.
- Emotki: tag `emotes=id:od-do` liczy pozycje w CODE POINTACH Unicode, nie w charach C#.
- Emoji nie ma w zadnym typowym foncie gry - wycinac (zakresy >=0x1F000, 0x2600-0x27BF,
  variation selectors, ZWJ), inaczej puste kwadraty.
- justinfan to mechanizm nieoficjalny (tolerowany od 10+ lat, uzywa go Chatterino/tmi.js);
  EventSub NIE MA trybu anonimowego. Funkcja musi degradowac sie gracefully (retry + status).
- Kick (stan 2026-07): oficjalne API dostarcza czat TYLKO webhookiem na publiczny adres
  (desktop bez serwera = nie da sie); nieoficjalnie dziala websocket Pusher bez auth,
  ale klucz/kanaly rotuja bez ostrzezenia, a lookup id kanalu blokuje Cloudflare (JA3).

## Kiedy uzyc
Kazda funkcja "czat steruje gra" w grze desktopowej single-player: darmowa, bez logowania,
bez infrastruktury. NIE nadaje sie, gdy trzeba PISAC na czacie (wymaga OAuth) albo
gwarantowanego SLA (mechanizm nieoficjalny).
