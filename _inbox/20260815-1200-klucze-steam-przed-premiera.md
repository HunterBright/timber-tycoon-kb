---
title: Zwykłe klucze Steam wysłane przed premierą = twórca nie zagra
type: anti-pattern
status: draft
confidence: medium
verified: ''
date: 2026-08-15
project: Kerf - Sawmill Tycoon
tags: [steam, wydanie, klucze, marketing, tworcy]
applies_to: []
source: ''
suggested-category: workflow/general
---

# Zwykłe klucze Steam wysłane przed premierą = twórca nie zagra

## Co się dzieje

Przed premierą kusi, żeby poprosić w panelu Steamworks o "klucze" i rozesłać je twórcom
i prasie. Domyślny rodzaj (**Default Release**) aktywuje się na koncie, ale gra odblokuje
się dopiero w dniu premiery. Twórca aktywuje klucz, widzi grę w bibliotece i **nie może
jej pobrać**. Efekt: stracony kontakt, bo drugi raz nikt nie odpisze.

## Jak jest dobrze

Do prasy i twórców przed premierą służy wyłącznie **Release State Override** - klucz,
który odblokowuje grę mimo braku premiery. Limit ok. 2500 sztuk łącznie.
Tych kluczy **nigdy nie wolno sprzedawać** (zasada Valve, nie sugestia).

## Dwie rzeczy, które trzeba zaplanować z wyprzedzeniem

1. **Trzy tygodnie od założenia AppID** - przy pierwszej grze na Steamie wcześniej nie da
   się nawet poprosić o klucze. To trzeba wliczyć w harmonogram premiery, nie odkryć w dniu wysyłki.
2. **Prośba idzie do przeglądu u Valve** i nie ma podanego terminu - bywa kilka godzin,
   bywa kilka dni. Nie planuj wysyłki kluczy jako zadania "na dziś wieczór".

## Warunek, o którym łatwo zapomnieć

Klucz odblokowuje grę, ale twórca pobiera to, co leży w gałęzi `default` na Steamie.
**Najpierw czysty build na SteamPipe, dopiero potem klucze** - inaczej klucz działa,
a pobierać nie ma czego.

## Powiązane

- [[20260815-1205-steam-auto-cloud-sciezka-zapisow|Zapisy w chmurze na Steamie: Auto-Cloud zamiast przepisywania systemu zapisu]] - wspolne: steam, wydanie
