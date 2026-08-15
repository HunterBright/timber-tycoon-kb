---
title: W sesji decyzyjnej wcześniejszy werdykt unieważnia późniejsze pytanie — pytaj o żywą resztę
type: pattern
status: draft
confidence: high
verified: ''
date: 2026-08-14
project: Another Quest
tags:
- gate
- decision-sessions
- workflow
- prompting
applies_to: []
source: 'GATE B8 — 30 pytań, z czego cztery uległy zmianie w trakcie sesji'
suggested-category: workflow/patterns
---

# W sesji decyzyjnej wcześniejszy werdykt unieważnia późniejsze pytanie — pytaj o żywą resztę

## Kontekst

Sesja decyzyjna z listą pytań przygotowaną **wcześniej**, przez inną sesję: 30 pozycji, każda z opcjami
i kosztami. Lista była zdeduplikowana i dobrze zrobiona — a i tak w trakcie sesji **cztery pytania
przestały znaczyć to, co znaczyły na papierze**, bo rozstrzygnęły je odpowiedzi na pytania wcześniejsze.

## Wzorzec

Po każdym werdykcie sprawdź pozostałe pytania i zaklasyfikuj każde do jednej z czterech grup:

| Grupa | Co zrobić |
|---|---|
| **Bezprzedmiotowe** — werdykt rozstrzygnął je w całości | Nie pytaj. Zapisz jako „zamknięte przez werdykt N", nie jako „odpowiedziane". To dwie różne rzeczy: inaczej ktoś będzie ścigał martwe zadanie |
| **Zawężone** — została żywa część, węższa niż pytanie | Zadaj **tylko żywą część** i powiedz, co ją zawęziło |
| **Wyprowadzalne** — odpowiedź wynika mechanicznie ze złożenia dwóch werdyktów | Nie pytaj, ale **zapisz jawnie jako wyprowadzone**, w osobnej sekcji. To jest miejsce, w którym decydent może Cię poprawić |
| **Nowe** — werdykt stworzył pytanie, którego nie było na liście | Zadaj. Sprawdź jego termin: pytanie stworzone przez werdykt często dziedziczy termin tego werdyktu |

## Dlaczego to działa

**Zadanie pytania rozstrzygniętego wcześniej jest gorsze niż strata czasu.** Decydent odpowiada
ponownie, czasem inaczej — i sesja kończy się dwoma sprzecznymi werdyktami na tę samą rzecz,
oba z jego podpisem. Wtedy nie ma już jak ustalić, który obowiązuje.

Symetrycznie: **wniosek wyprowadzony i niezapisany jako wyprowadzony** wygląda później identycznie
jak werdykt, który padł. W projekcie z historią rozjeżdżania się dokumentów to jest dokładnie ten
sposób, w jaki powstają cytaty bez pokrycia.

## Przykład

Pytanie brzmiało: *„gracz martwy przy wyjściu drużyny portalem — co ma w rozliczeniu?"*, z trzema opcjami
architektonicznymi. Wcześniejszy werdykt o rozłączeniu ustalił już, że powód zakończenia opisuje **drużynę**,
a stan pojedynczego gracza idzie **flagą** — czyli wybrał architekturę.

Zadana została wyłącznie żywa reszta, której na liście nie było: dwa werdykty zamówiły **dwie różne flagi
na tej samej osi** („nieobecny" i „żył w chwili wyjścia"), a gracz może być martwy **i** rozłączony naraz.
Pytanie „jedno pole o trzech wartościach czy dwie flagi?" jest realną decyzją o kształcie zapisu gry —
a powstało dopiero w trakcie sesji.

Osobno sprawdzono u źródła, czy druga część pierwotnego pytania (wypłata dla martwego) jest w ogóle
otwarta. Nie była — errata mówiła „wyłącznie dla wyjścia żywym", więc litera i intencja wskazywały to samo.
**Pytanie, które plik uznał za otwarte, było już rozstrzygnięte w innym dokumencie.**

## Koszt i granice

Wymaga, żeby prowadzący sesję **trzymał w głowie stan wszystkich pozostałych pytań** — czyli żeby był
modelem mocnym i miał w kontekście całą listę. Przy delegowaniu pytań do osobnych wykonawców ten wzorzec
nie zadziała: wykonawca widzi swoje pytanie, nie sesję.

Ryzyko odwrotne: **zawężanie może przejść w projektowanie.** Zabezpieczenie — zanim zawęzisz, sprawdź,
czy odrzucana część naprawdę została rozstrzygnięta, i **napisz w zapisie, przez który werdykt**.
Jeśli nie potrafisz wskazać numeru, to nie jest zawężenie, tylko Twoja własna decyzja.

## Related

- [[pokaz-material-i-zasade-nie-sam-wynik]]
- [[wyluskaj-niezmiennik-zanim-obalisz-decyzje]]
