---
title: Zamówienie losowane OD KWOTY w dół - cennik przestaje cokolwiek znaczyć
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- economy
- tycoon
- pricing
- orders
- progression
- design
applies_to: []
source: ''
suggested-category: genre/anti-patterns
---

# Zamówienie losowane OD KWOTY w dół - cennik przestaje cokolwiek znaczyć

## The trap

Generator zamówień w grze ekonomicznej/tycoonowej robi to, co wydaje się najprostsze i najbardziej
kontrolowalne:

1. wylosuj DOCELOWĄ KWOTĘ zamówienia z widełek zależnych od poziomu gracza (np. 100-135 zł),
2. dobieraj produkty i sztuki, aż suma zbliży się do tej kwoty,
3. zapłać dokładnie tyle, ile wyszło.

Wygląda na idealne narzędzie balansu: projektant steruje przychodem wprost, jedną tabelą widełek,
i ma gwarancję, że gracz nie dostanie zamówienia poza skalą.

## Why it fails

**Cena jednostkowa przestaje wpływać na zarobek. Wpływa wyłącznie na to, ILE SZTUK trzeba przynieść.**

Jeśli gracz podniesie jakość surowca (droższe drewno, lepszy tier produktu), gra po prostu **poprosi
o mniej sztuk za tę samą kwotę**. Zarobek na zamówienie się nie zmienia.

Konsekwencje, wszystkie ciche:

- **Cała drabina wartości surowców jest ozdobą.** 10 gatunków drewna z cenami 16-72 zł to dekoracja.
- **Praca nie przyspiesza gry.** Zarobek jest funkcją POZIOMU, nie wysiłku. Gracz, który zdobył
  najdroższy surowiec, zarabia tyle samo co ten, który został przy najtańszym - tylko mniej dźwiga.
  (Paradoksalnie: droższy surowiec to *wygoda*, nie *bogactwo*.)
- **Progresja odkleja się od gospodarki.** Skoro przychód zależy tylko od poziomu, to poziom musi
  być czymś bramkowany - zwykle reputacją. I nagle prawdziwym hamulcem gry jest reputacja, mimo że
  projekt zakładał, że hamulcem są pieniądze.
- **Podnoszenie cen w ramach balansu NIC NIE ROBI.** To najbardziej mylący objaw: projektant stroi
  cennik całymi dniami i nie widzi żadnej zmiany w tempie gry.

## Symptoms

- W kodzie: `targetValue = Random.Range(minOrderValue, maxOrderValue)`, potem pętla dosypująca sztuki
  do `targetValue`, i na końcu `reward = suma(sztuki x cena)`.
- Tabela "widełek kwot per poziom" jest jedynym miejscem, które realnie steruje przychodem.
- W grze są dwa kanały sprzedaży i liczą pieniądze DWIEMA RÓŻNYMI METODAMI (drugi kanał zwykle liczy
  poprawnie: cena x ilość). Kanały nie dają się ze sobą zestroić i nikt nie wie dlaczego.
- Symulacje ekonomii nie zgadzają się z odczuciem z gry.

## Correct approach

**Losuj LICZBĘ SZTUK, nie kwotę.** Klient mówi "poproszę 6 desek" (i ewentualnie jakich), a wypłata
= suma(sztuki x cena z cennika).

Wtedy:
- droższy surowiec = **więcej pieniędzy za to samo dźwiganie** (praca realnie przyspiesza grę),
- cennik znów jest narzędziem balansu,
- oba kanały sprzedaży liczą tak samo i dają się zestroić jedną tabelą cen.

Sterowanie tempem przenosi się na rzeczy, które nadal masz pod kontrolą i które są UCZCIWE:
- liczba sztuk w koszyku (rośnie POWOLI: 4 -> 11 przez całą grę),
- liczba klientów na dobę (i coś, co ją realnie podnosi - szyld, reklama),
- ceny ulepszeń.

Efekt: wypłata rośnie szybko (60 -> 700), ale **nie dlatego, że gra tak postanowiła - tylko dlatego,
że gracz przeszedł na droższe drewno.** Cena sztuki nie jest ustawiana przez grę, ona WYNIKA z tego,
co gracz trzyma na regale.

## Uwaga na pułapkę towarzyszącą

Jeśli pula losowania zamówień jest ważona tym, co **rośnie w świecie** (stojące drzewa), zamiast tym,
co gracz **ma na regale**, to sadzenie drzew zaczyna graczowi SZKODZIĆ: wycięcie taniej strefy do gołej
ziemi kasuje tani towar z rynku. Optymalny gracz zamienia las w kopalnię odkrywkową. Waż popytem po
STANIE MAGAZYNU gracza; świat trzymaj wyłącznie jako bezpiecznik anty-zator ("nie proś o coś, czego
fizycznie nie da się zdobyć").

## Case study (Timber Tycoon, 2026-07-14)

Gra miała 10 gatunków drewna (deski 16-44 zł), 13 poziomów tartaku i pełny łańcuch przetwórstwa.
Cennik nie wpływał na nic. Drugi kanał (meble) liczył poprawnie (cena desek x jakość) i przez to
zarabiał wielokrotnie więcej niż lada - dwa kanały żyły w dwóch różnych światach cenowych i nigdy
nie dało się ich zestroić. Wykryte dopiero przy pełnym audycie ekonomii, po roku prac.
