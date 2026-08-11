---
title: Blokada linków na Discordzie działa tylko przez AutoMod, nie przez uprawnienia
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-10
project: Discord_Studio
tags:
- discord
- automod
- bezpieczenstwo
- regex
- uprawnienia
applies_to: []
source: 'Sesja budowy makiety serwera MGDB Studio'
severity: high
suggested-category: tooling/lessons
time_lost: ''
---

# Blokada linków na Discordzie działa tylko przez AutoMod, nie przez uprawnienia

## Problem

Standardowy wzorzec „linki tylko dla zaufanych" buduje się przez odebranie roli `@everyone`
uprawnienia `EmbedLinks` i nadanie go roli zaufanej. Wygląda to na zabezpieczenie przed
oszustwami i tak bywa opisywane w poradnikach.

**Nie jest.** `EmbedLinks` steruje wyłącznie tym, czy pod wiadomością pojawi się podgląd strony.
Sam adres nadal wchodzi jako zwykły tekst i nadal jest klikalny. Konto tworzone przez oszusta
wkleja link do fałszywego Steama i uprawnienia go nie zatrzymują.

Drugi problem: reguła AutoMod przestaje być czytelna, gdy odwołuje się do skasowanej roli.
Po usunięciu ról z listy wyjątków odczyt reguł przez API wywalał się na
`Cannot read properties of undefined (reading 'name')`, mimo że same reguły działały.

## Root cause

`EmbedLinks` to uprawnienie prezentacji, nie treści. Discord nie ma żadnego uprawnienia
kanałowego, które blokuje wysłanie adresu URL. Jedyne narzędzie, które patrzy na treść
wiadomości, to AutoMod.

Awaria odczytu reguł bierze się stąd, że lista wyjątków przechowuje identyfikatory ról.
Po skasowaniu roli identyfikator zostaje jako martwe odwołanie, a warstwa, która zamienia je
na nazwy, przewraca się na nieistniejącym obiekcie.

## Solution

**Blokada linków:** reguła AutoMod typu `keyword` z wyrażeniem `(?i)https?://[^\s]+`, akcja
`block`, plus `alert` na osobny kanał kwarantanny. Role zaufane trafiają do wyjątków.
Uzupełniająco wzorce na `www.` bez protokołu i na domeny najwyższego poziomu używane głównie
przez oszustów (`.ru`, `.tk`, `.xyz`, `.top`, `.click`, `.zip`, `.mov`).

**Biała lista domen** ustawiana jest w polu `allow_list` reguły. Wiele mostów i bibliotek tego
pola nie wystawia — wtedy zostaje wpisanie ręczne w interfejsie Discorda.

**Osobny kanał na alerty od linków**, nie wspólny z resztą moderacji. Ten kanał ma dwa zadania,
oba wymagające czytania jednym rzutem oka: wyłapać uczciwe linki zjedzone przez filtr (idą na
białą listę) i wyłapać to samo konto trzeci raz (to już bot na skrypcie, nie pomyłka).

**Po skasowaniu roli** przejrzeć wszystkie reguły AutoMod i wyczyścić martwe odwołania w polach
wyjątków. Robi się to jednym `edit` na regułę z poprawioną listą.

## What didn't work

- **`EmbedLinks` i `AttachFiles` jako brama antyoszustwowa.** Kosmetyka. Zatrzymuje podgląd,
  przepuszcza adres.
- **Wyrażenia z negatywnym wyprzedzeniem**, np. `(steam|discord)[a-z0-9-]*\.(?!com\b)[a-z]{2,}`,
  żeby złapać podróbki domen. Discord używa składni Rusta, która nie ma `(?!...)`. API odrzuca
  taki wzorzec komunikatem `AUTO_MODERATION_REGEX_SYNTAX`. Zamiast tego wypisać podróbki wprost
  jako słowa kluczowe: `steamcomunity`, `dlscord`, `discordgift`, `disc0rd`.
- **Akcja `timeout` w regule**, gdy bot nie ma uprawnienia `ModerateMembers`. Cała reguła zostaje
  odrzucona, mimo że sam wyzwalacz jest poprawny. Dodać akcję dopiero po nadaniu uprawnienia.

## Transferability

Dotyczy każdego serwera Discorda z otwartą rejestracją, niezależnie od projektu. Wniosek szerszy:
**przy warstwie bezpieczeństwa sprawdzić, co dane uprawnienie faktycznie blokuje, a nie co
sugeruje jego nazwa.** `EmbedLinks` brzmi jak kontrola linków i nią nie jest. To samo pytanie
warto zadać każdemu przełącznikowi opisanemu rzeczownikiem.

## Related
- [[20260810-1745-discord-bot-viewchannel-samoodciecie]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-1950-honeypot-kanal-pulapka-na-boty|20260810-1950-honeypot-kanal-pulapka-na-boty]] - wspolne: automod, bezpieczenstwo, discord
- [[20260810-1745-discord-bot-viewchannel-samoodciecie|Bot Discorda bez Administratora nie ukryje kanału przed @everyone]] - wspolne: uprawnienia, discord
<!-- /POWIAZANE:auto -->
