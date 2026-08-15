---
title: Bezpiecznik wymienia narzedzia, a zagrozenie chodzi drogami - poszerz wszystkie matchery naraz
type: lesson
status: draft
confidence: high
verified: 2026-08-15
tags: [claude-code, hooki, bezpieczenstwo, mcp, automatyzacja]
date: 2026-08-15
project: GameDevOS
source: warsztat 2026-08-14 i 2026-08-15, dwa niezalezne wystapienia
applies_to: [claude-code, hooki, serwery-mcp, konfiguracja-globalna]
suggested-category: engine/lessons
severity: high
time_lost: dwa przebiegi warsztatu, dwa dni
---

# Bezpiecznik wymienia narzedzia, a zagrozenie chodzi drogami

## Objaw

Hook w `settings.json` mial poprawna tresc, przechodzil wlasne testy i **mimo to
nie chronil przed niczym**, co szlo droga, ktorej jego pole `matcher` nie znalo.

To sie zdarzylo **dwa razy w dwa dni, w dwoch roznych bezpiecznikach**:

1. **14.08.2026** - straznik zapisu do projektu gry mial
   `matcher: Edit|Write|NotebookEdit|MultiEdit`. Serwery MCP do Unity i Blendera
   pisza do projektu **wlasnymi** narzedziami (`mcp__coplay-mcp__*` i podobne)
   i szly obok blokady. Wykryte dopiero wtedy, gdy agent sam poprosil Unity
   o zapis zrzutu do repozytorium gry.
2. **15.08.2026** - ochrona tokenu konta (`.credentials.json`) miala
   `matcher: Bash|Read|Grep`. **Ten sam mechanizm, ten sam plik konfiguracji,
   naprawiony dzien wczesniej w sasiednim wpisie** - i nikt go nie ruszyl.

## Przyczyna

Hook z waskim `matcher` wyglada w konfiguracji **dokladnie tak samo** jak hook,
ktory dziala. Nie ma zadnego objawu: pliki sa na miejscu, testy skryptu
przechodza, `--sprawdz-sie-samemu` swieci na zielono. Skrypt hooka jest testowany
na danych, ktore mu ktos **poda recznie** - a w prawdziwym zyciu nie zostaje
w ogole zawolany, bo harness dopasowuje nazwe narzedzia do wzorca **zanim**
skrypt dostanie cokolwiek na wejscie.

Drugie wystapienie jest wazniejsze od pierwszego, bo pokazuje, ze poprawka
byla **za waska**: naprawiono jeden hook zamiast przejrzec wzorzec. Dziura
siedziala w sposobie myslenia („co to narzedzie robi"), a nie w pliku
(„ktore narzedzia wymienic").

## Rozwiazanie

Przy kazdym bezpieczniku zadaj **dwa** pytania, nie jedno:

1. Czy jego tresc jest poprawna?
2. **Czy on w ogole zostanie zawolany dla tej drogi?**

Konkretnie:

- **Mysl drogami, nie narzedziami.** Nie „blokuje `Write`", tylko „blokuje
  kazda droge, ktora tworzy albo zmienia plik". Kazdy serwer MCP, kazda wtyczka,
  kazde nowe narzedzie to **nowa droga**, ktorej stary matcher nie zna.
- **Przy poszerzaniu jednego matchera przejdz od razu wszystkie pozostale hooki
  w pliku.** Jesli jeden bezpiecznik mial za waski wzorzec, pozostale powstaly
  w tym samym mysleniu i prawie na pewno maja ten sam blad.
- Dla ochrony przed czyms, czego **nigdy** nie wolno dotknac, uzyj wzorca
  otwartego (`mcp__.*`, a nie lista serwerow). Koszt to jeden dodatkowy `grep`
  na wywolanie; zysk to odpornosc na serwer, ktory dojdzie za pol roku.
- Dla ochrony przed **zapisem** odwroc domysl: blokuj wszystko poza jawna lista
  narzedzi czytajacych. Lista czytajacych jest krotka i znana; lista piszacych
  rosnie z kazda aktualizacja cudzego serwera.

## Co NIE zadzialalo

- **Testowanie skryptu hooka podanymi recznie danymi.** Sprawdza tresc, nie
  dosiegalnosc. Skrypt moze byc bezbledny i nigdy nie zostac uruchomiony.
- **Naprawienie jednego hooka i uznanie tematu za zamkniety** (14.08). Sasiedni
  wpis w tym samym pliku mial ten sam blad i przelezal jeszcze dobe.
- **Poleganie na tym, ze serwer MCP „chyba nie ma takiego narzedzia".** Serwery
  wystawiaja po kilkadziesiat narzedzi, w tym `read_file`, `list_files`,
  `execute_*_code`. Ostatnie z nich wykonuje dowolny kod, czyli potrafi
  wszystko, niezaleznie od tego, jak sie nazywa.

## Dowod

Dzwignia lamiaca uruchomiona na **zywym** narzedziu MCP, nie na atrapie:

- wywolanie serwera MCP z nazwa chronionego pliku w argumencie -> **zablokowane**
  (przed poszerzeniem matchera przechodzilo swobodnie);
- to samo narzedzie z uczciwym zapytaniem -> **przeszlo**, czyli poszerzenie nie
  zamurowalo normalnej pracy.

Test negatywny jest tu rownie wazny jak pozytywny: bezpiecznik, ktory blokuje
wszystko, zostanie wylaczony przez czlowieka w ciagu tygodnia.

Zapis obu wystapien: `D:\GameDevOS\90-Warsztat\DZIENNIK\2026-08-14.md`
i `2026-08-15.md`; rejestr: `90-Warsztat\WDROZENIA.md`.

## Czy to przeniesie sie na inny projekt

**Tak, i to poza Claude Code.** To jest ogolna wlasnosc kazdej listy kontroli
dostepu opartej na **wymienianiu bytow** zamiast na **opisywaniu zdolnosci**:
regul zapory sieciowej, uprawnien w CI, `allowlist` w skryptach zwalniajacych.
Lista bytow starzeje sie w chwili, gdy dochodzi nowy byt; opis zdolnosci nie.

Cecha rozpoznawcza, po ktorej warto to wylapac u siebie: **czy dopisanie nowego
narzedzia albo integracji wymaga recznego dopisania go do bezpiecznika?**
Jesli tak, bezpiecznik jest juz nieaktualny - po prostu jeszcze o tym nie wiesz.

## Powiazane

- [[straznik-zapisu-mcp]]
- [[bezpiecznik-ktory-milczy-fail-open]]
