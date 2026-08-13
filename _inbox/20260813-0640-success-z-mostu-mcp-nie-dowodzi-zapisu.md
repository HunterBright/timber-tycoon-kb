---
title: "\"success\" z mostu MCP nie dowodzi, że usługa przyjęła zmianę"
type: lesson
status: draft
confidence: high
verified: '2026-08-13 — odczyt stanu i dziennik serwera Discorda'
date: 2026-08-13
project: Discord_Studio (MGDB Studio)
tags:
- mcp
- api
- weryfikacja
- discord
applies_to: []
source: ''
severity: high
suggested-category: workflow/lessons
time_lost: '~10 min, ale mogło być kilka dni'
---

# „success" z mostu MCP nie dowodzi, że usługa przyjęła zmianę

## Problem
Wgranie banera serwera przez most MCP do Discorda zwróciło:

```json
{ "success": true, "message": "Server images updated successfully", "updated": ["banner"] }
```

Odczyt stanu chwilę później pokazał `"banner": null`. W dzienniku serwera **nie było po tej
zmianie żadnego wpisu** — nie „zmiana odrzucona", tylko brak zdarzenia. Gdyby nie sprawdzenie,
raport brzmiałby „baner wgrany", a serwer stałby bez banera.

## Root cause
Baner serwera wymaga **2. poziomu ulepszeń**, a serwer miał 0. Discord nie przyjął tej części
żądania. Most raportuje **powodzenie wysłania żądania**, a nie powodzenie zmiany po stronie
usługi — jego `success` mówi „nie wyleciał wyjątek", nie „stan się zmienił".

Ten sam wzorzec siedzi w każdym wrapperze, który tłumaczy wywołanie narzędzia na wywołanie
API: wrapper zna kod odpowiedzi, nie zna semantyki usługi.

## Solution
Po każdej zmianie stanu przez most: **odczytaj stan z powrotem** i porównaj z zamiarem.
Tam, gdzie usługa prowadzi dziennik zdarzeń (Discord, GitHub, Jira), dziennik jest dowodem
mocniejszym niż odczyt — pokazuje, czy zdarzenie w ogóle zaistniało.

Gdy odczyt przeczy odpowiedzi, warto **rozdzielić przyczyny osobnym eksperymentem**. Tutaj
nie było wiadomo, czy wina leży po stronie limitu usługi, czy po stronie mostu (może nie
umie czytać lokalnych ścieżek). Ponowne wgranie **tej samej ikony, która już stała** —
zmiana bezpieczna i odwracalna — pokazało zmianę hasza w dzienniku, czyli ścieżki lokalne
działają, a winny jest limit poziomu ulepszeń.

## What didn't work
Zaufanie polu `updated: ["banner"]`. Brzmi jak potwierdzenie od usługi, a jest echem tego,
o co narzędzie zostało poproszone.

## Transferability
Dotyczy każdego narzędzia agenta, które pisze do zewnętrznej usługi: MCP, wrappery REST,
klienty CLI opakowane w skrypt. Reguła: **zmiana stanu nie jest zrobiona, dopóki nie
odczytano jej z powrotem** — a przy raportowaniu użytkownikowi opisuj to, co pokazał odczyt,
nie to, co odpowiedziało narzędzie.

Szczególnie groźne przy limitach powiązanych z płatnym poziomem konta (poziomy ulepszeń,
plany, limity API), bo tam usługa często odrzuca cicho, zamiast zwrócić błąd.

## Related
- [[20260813-0745-podglad-w-docelowej-ramce-zamiast-oceny-pelnego-kadru]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260810-1745-discord-bot-viewchannel-samoodciecie|Bot Discorda bez Administratora nie ukryje kanału przed @everyone]] - wspolne: discord, mcp
<!-- /POWIAZANE:auto -->
