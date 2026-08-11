---
type: pattern
project: Discord_Studio
suggested-category: tooling/patterns
tags:
- discord
- bezpieczenstwo
- boty
- honeypot
- automod
date: 2026-08-10
status: draft
---

# Kanał-pułapka na boty spamerskie (honeypot) na Discordzie

## Problem, który rozwiązuje

Filtry treści łapią to, **co** bot napisał. Nie łapią tego, **że** to bot. Konto spamerskie,
które napisze coś niewinnego, przechodzi przez AutoMod bez zaczepienia, a potem zaczyna
działać na priv, gdzie AutoMod w ogóle nie sięga.

Honeypot łapie po zachowaniu, nie po treści. Bot spamerski działa według schematu:
wejdź na serwer, wypisz się w każdym kanale, w którym się da. Człowiek tak nie robi.

## Jak to działa

Kanał widoczny dla `@everyone`, **otwarty do pisania**, z wielkim ostrzeżeniem w treści:
„nie pisz tutaj, każda wiadomość to natychmiastowy ban". Bot ostrzeżenia nie czyta i pisze.
Ban leci w sekundę, zanim bot dojdzie do prawdziwych kanałów.

Zaobserwowane na serwerze Kroniki Myrtany (17 tys. członków): licznik w opisie kanału pokazywał
**57 złapanych kont**. Robi to tam aplikacja o nazwie **Honeypot**.

## Warunki, bez których to nie działa

**1. Kanał musi być wyjęty spod wszystkich reguł AutoMod.**
To jest cały sekret i łatwo go przeoczyć. Jeśli AutoMod zablokuje wiadomość bota, to bot jej
**nie wysłał** — nie ma zdarzenia, nie ma czego złapać, bot idzie dalej nietknięty.
Blokada treści i pułapka na zachowanie wykluczają się w tym jednym kanale.
Ustawia się to w polu `exempt_channels` każdej reguły.

**2. Kanał musi być otwarty do pisania dla niezweryfikowanych.**
Jeśli serwer ma bramkę wejściową (Onboarding, rola po weryfikacji), pułapka musi być
**po niewłaściwej stronie bramki** — dostępna zanim ktokolwiek cokolwiek kliknie.
W przeciwnym razie łapie tylko boty, które przeszły bramkę, czyli te lepsze.

**3. Ostrzeżenie w językach społeczności.**
Jedyny koszt tego rozwiązania to człowiek, który napisze tam z ciekawości. Ostrzeżenie ma być
krzyczące, na górze, przypięte i w każdym języku, jakim mówi serwer.

## Ciekawy efekt uboczny

Przy serwerze z bramką wejściową honeypot bywa **jedynym kanałem, w którym świeże konto
w ogóle może pisać**. Wtedy pułapka nie jest jednym z wielu miejsc — jest jedynym wyjściem,
a jej skuteczność zbliża się do stu procent dla botów działających schematem.

## Czego nie rozwiązuje

Wiadomości prywatnych. Bot, który wchodzi cicho i od razu pisze na priv do członków, nigdy nie
tknie pułapki. Na to potrzebna jest osobna warstwa wykrywająca wzorce wchodzenia
(na Discordzie: Beemo, Wick).

## Related
- [[20260810-1930-discord-automod-regex-i-blokada-linkow]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-1930-discord-automod-regex-i-blokada-linkow|Blokada linków na Discordzie działa tylko przez AutoMod, nie przez uprawnienia]] - wspolne: automod, bezpieczenstwo, discord
<!-- /POWIAZANE:auto -->
