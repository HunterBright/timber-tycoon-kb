---
title: Bot Discorda bez Administratora nie ukryje kanału przed @everyone
type: lesson
status: draft
confidence: high
verified: ''
date: 2026-08-10
project: Discord_Studio
tags:
- discord
- bot
- uprawnienia
- mcp
- automatyzacja
applies_to: []
source: 'Sesja budowy makiety serwera MGDB Studio na serwerze Kerf'
severity: high
suggested-category: tooling/lessons
time_lost: '~40 min'
---

# Bot Discorda bez Administratora nie ukryje kanału przed @everyone

## Problem

Bot z drobiazgowo dobranymi uprawnieniami (ManageChannels, ManageRoles, ViewChannel — bez
Administratora) tworzył kategorie, kanały i role bez problemu. Ale każda próba odebrania roli
`@everyone` uprawnienia `ViewChannel` na kanale lub kategorii kończyła się błędem `Missing
Permissions` (50013). Ta sama operacja dla dowolnego innego uprawnienia — `SendMessages`,
`EmbedLinks`, `AttachFiles`, `MentionEveryone` — przechodziła bez oporu.

Druga twarz tego samego problemu, groźniejsza, bo cicha: po zablokowaniu `EmbedLinks` dla
`@everyone` na kategorii bot **stracił to uprawnienie sam dla siebie** w całej tej gałęzi.
Skutek: nie mógł już wysłać wiadomości z embedem do kanału w tej kategorii ani nadać tego
uprawnienia komukolwiek innemu. Błąd wygląda identycznie jak problem z hierarchią ról, więc
diagnoza idzie w złą stronę.

## Root cause

Discord liczy uprawnienia w kanale tak: uprawnienia bazowe (suma `@everyone` i wszystkich ról
członka) → nadpisania dla `@everyone` → nadpisania dla ról → nadpisania dla członka.

Nadpisanie `deny` na `@everyone` działa na **wynik sumy bazowej**, więc kasuje uprawnienie
także tym, którzy mieli je z własnej roli — w tym botowi. Bot nie ma nadpisania roli ani
członka, które by mu je przywróciło.

Nakłada się na to reguła API: **można zmieniać wyłącznie te bity uprawnień, które samemu się
posiada w danym kanale**. Stąd zamknięte koło:

1. Bot blokuje `X` dla `@everyone` → traci `X` w tym kanale.
2. Bot chce nadać `X` roli Member → odmowa, bo już nie ma `X`.

Przy `ViewChannel` Discord odmawia od razu, na pierwszym kroku — chroni aplikację przed
odcięciem sobie dostępu do kanału, którym ma zarządzać.

## Solution

Dwie drogi:

1. **Administrator na czas budowy.** Nadać botowi uprawnienie Administrator, zbudować całą
   warstwę uprawnień, odebrać po robocie. Administrator omija liczenie nadpisań w całości.
   W praktyce jedyna droga, jeśli struktura ma używać ukrytych kanałów.
2. **Odwrócenie modelu.** Odebrać `ViewChannel` roli `@everyone` na poziomie serwera i
   otwierać kategorie jawnymi `allow`. Nie wymaga żadnego `deny`, więc pułapka nie występuje.
   Kosztowne: trzeba wyczyścić `ViewChannel` z **każdej** roli, która je odziedziczyła.

**Kolejność operacji jest kluczowa niezależnie od drogi.** Najpierw nadać wyjątki (`allow`)
rolom, które mają dostęp zachować, dopiero na końcu zakładać blokadę `deny` dla `@everyone`.
Odwrotna kolejność zamyka drogę powrotną.

## What didn't work

- **Nadpisanie dla własnej roli bota.** Odmowa — bot nie może zarządzać rolą, która jest jego
  najwyższą rolą. Rola bota nie jest "poniżej samej siebie".
- **Druga rola dla bota z jawnym `allow` na `ViewChannel`.** Sprawdzone i **nie działa**.
  Nadanie botowi dodatkowej roli (tu: roli `Admin`) i danie tej roli nadpisania
  `allow: ViewChannel` na kategorii nie odblokowuje blokady `deny: ViewChannel` dla
  `@everyone` — Discord odmawia dalej. Ochrona przed samoodcięciem nie patrzy na to, czy
  dostęp faktycznie zostanie zachowany innym kanałem. Nie tracić na to czasu.
- **Nadawanie bitów zarządczych w nadpisaniu.** `ManageChannels`, `ManageRoles` i
  `ManageMessages` w `allow` wywracają całe wywołanie, mimo że bot je posiada. Discord wymaga
  na to Administratora. Usunięcie ich z listy przepuszcza resztę.
- **Nadpisanie dla bota jako członka.** Most MCP nie potrafi rozwiązać bota jako członka
  (`Member not found`), mimo że `list_members` pokazuje go poprawnie. Bota nie da się też przez
  most dodać do roli technicznej z punktu 2 — musi to zrobić człowiek w interfejsie Discorda.
- **Edycja nadpisania na kategorii licząc na propagację w dół.** Kanał utworzony w kategorii
  dostaje **własną kopię** nadpisań w chwili powstania. Późniejsza zmiana kategorii go nie
  dotyka. Trzeba poprawiać kanał po kanale.
- **Czytanie komunikatu błędu wprost.** Most podpowiada "podnieś rolę bota wyżej w hierarchii",
  co jest myląco nietrafne — rola bota była już najwyższa na serwerze.

## Transferability

Dotyczy każdego projektu, w którym bot lub skrypt buduje strukturę serwera Discorda — niezależnie
od gatunku gry i silnika. Reguła "można nadawać tylko to, co się ma" obowiązuje w całym API
Discorda, więc ta sama pułapka wraca przy automatyzacji moderacji, ticketów i ról z reakcji.

Szerszy wniosek poza Discordem: **przy automatyzacji systemu uprawnień sprawdzić najpierw, czy
narzędzie potrafi wyłączyć samo siebie.** Jeśli tak, kolejność operacji przestaje być kwestią
wygody i staje się warunkiem wykonalności.

## Related
- Portal Discorda odrzuca całą paczkę zmian, gdy jedna jest nieprawidłowa — zmieniać po jednej

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-1930-discord-automod-regex-i-blokada-linkow|Blokada linków na Discordzie działa tylko przez AutoMod, nie przez uprawnienia]] - wspolne: uprawnienia, discord
- [[20260813-0800-panel-saas-bez-api-zjada-automatyzacje|Panel SaaS bez API zjada automatyzację przeglądarkową — rozpoznaj to po trzech objawach]] - wspolne: discord, automatyzacja
- [[20260813-0640-success-z-mostu-mcp-nie-dowodzi-zapisu|"success" z mostu MCP nie dowodzi, że usługa przyjęła zmianę]] - wspolne: discord, mcp
<!-- /POWIAZANE:auto -->
