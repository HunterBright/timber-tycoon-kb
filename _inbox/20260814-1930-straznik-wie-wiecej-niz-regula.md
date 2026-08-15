---
title: Zanim napiszesz wyjątek od reguły, przeczytaj strażnika — on często rysuje granicę, której reguła nie zna
type: lesson
status: draft
confidence: high
verified: '2026-08-14'
severity: medium
date: 2026-08-14
project: Another Quest
tags: [permissions, guard-rails, proces, regula-vs-egzekwowanie, tooling, deny-list]
applies_to: [claude-code, ci, linters, permission-systems, git-hooks]
source: 'sesja errat B8, 2026-08-14 — wariant C dla 00_MATRYCA_ZALEZNOSCI.md'
suggested-category: workflow/lessons
---

# Zanim napiszesz wyjątek od reguły, przeczytaj strażnika — on często rysuje granicę, której reguła nie zna

## Sytuacja

Projekt miał regułę procesu: *„do `docs/canon/` piszesz wyłącznie przez `tools/canon_apply.py`"*.
Okazało się, że jeden z plików w tym katalogu — tabela zależności czytana **wierszem** — nie daje się
tym narzędziem poprawić: narzędzie **dopisuje blok na końcu pliku** i nie zmienia treści.

Postawiłem to jako decyzję do podjęcia i wyceniłem trzy warianty. Wariantowi „uznać ten plik za osobną
klasę" przypisałem cenę: **„wyjątek w regule; każdy wyjątek trzeba potem pamiętać"**.

## Co się okazało przy wykonaniu

**Ta cena nie istniała.** Lista `permissions.deny` blokowała wyłącznie `*_LOCKED.md` i jeden konkretny
dokument — **plik nawigacyjny nigdy nie był chroniony**. Warstwa uprawnień rysowała dokładnie tę granicę,
którą właśnie „wprowadzaliśmy jako wyjątek", i robiła to od początku.

Decyzja nie tworzyła więc wyjątku. **Doganiała regułą to, co strażnik już robił.** To zmieniło ryzyko
z „poluzowanie ochrony" na „usunięcie rozjazdu" — czyli z decyzji spornej na decyzję oczywistą.

## Dlaczego to się dzieje

Reguła w dokumencie i mechanizm ją egzekwujący **powstają w innych momentach i przez inne osoby**.
Reguła jest pisana ogólnie („cały katalog"), bo tak się ją wygodnie formułuje. Strażnik jest pisany
konkretnie (glob, ścieżka, wzorzec nazwy), bo inaczej nie da się go napisać. **Ta konkretność zawiera
wiedzę, której ogólna reguła nie ma** — ktoś, kto pisał listę wzorców, musiał się zastanowić nad każdym
plikiem z osobna.

Kiedy reguła i strażnik się rozjeżdżają, **to zwykle reguła jest zbyt szeroka**, a nie strażnik dziurawy.
Odwrotny przypadek też istnieje i jest groźniejszy — ale rozpoznasz go dopiero, gdy porównasz oba.

## Reguła robocza

**Zanim zaproponujesz wyjątek od reguły procesu, przeczytaj kod, który tej reguły pilnuje.**
Trzy pytania, wszystkie tanie:

1. **Czy strażnik w ogóle pilnuje tego przypadku?** (lista `deny`, glob w linterze, ścieżki w CI, hook)
2. **Jeśli nie — czy to przeoczenie, czy świadoma granica?** Historia pliku strażnika zwykle odpowiada.
3. **Jeśli granica jest świadoma — twój „wyjątek" jest zapisaniem istniejącego stanu**, nie zmianą.
   Cena spada, a wraz z nią ciężar decyzji.

## Druga połowa lekcji: wyjątek zapisz w narzędziu, nie w pamięci

Skoro cena „trzeba pamiętać" była jedynym realnym kosztem wariantu, warto ją **zdjąć do zera**: klasa
została wpisana do narzędzia jako nazwana kategoria, widoczna w wyniku bramki, zamiast zostać zdaniem
w dokumencie procesu.

**Do tego obowiązkowy warunek: nowa klasa nie może wyciszać kontroli.** Rozróżnienie, które trzeba
napisać wprost i przetestować:

- klasa zmienia **KTO i CZYM wolno pisać**;
- klasa **NIE zmienia tego, czy zmiana jest zauważana**.

Sprawdzenie, że tak jest, ma być **osobną próbą w samoteście narzędzia**, a nie założeniem. Test „czy
próba w ogóle gryzie" robi się przez sabotaż kopii narzędzia (wyciszyć kontrolę dla nowej klasy →
samotest musi upaść z nazwaną przyczyną) — kopii, nigdy oryginału.

## Anty-wzorzec obok

Odwrotna kolejność — najpierw napisać wyjątek w dokumencie, potem zorientować się, że strażnik i tak go
nie egzekwował — daje **regułę opisującą stan, który nigdy nie obowiązywał**. Taki zapis jest gorszy niż
brak zapisu: czytelnik wierzy, że ochrona istniała i została świadomie zdjęta, więc nie sprawdza, czy
w ogóle kiedykolwiek działała.
