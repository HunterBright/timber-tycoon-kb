---
title: Minigra zużywa surowiec na STARCIE, a przerwanie go zwraca
type: anti-pattern
status: active
confidence: medium
verified: ''
date: '2026-07-14'
project: Kerf - Sawmill Tycoon
tags:
- economy
- minigame
- exploit
- balance
- abort
- refund
applies_to: []
source: ''
promoted: '2026-07-30'
---

# Minigra zużywa surowiec na STARCIE, a przerwanie go zwraca

## The trap

Minigra przetwarzania (kłoda -> deski, zrębki -> pelet) pobiera surowiec z magazynu w momencie
uruchomienia, bo "gracz już zaczął, więc surowiec jest w maszynie". Osobno, ścieżka przerwania
(ESC / wyjście z minigry) uprzejmie zwraca surowiec, "żeby gracz nie stracił materiału przez
przypadkowe wyjście". Każda z tych decyzji z osobna brzmi rozsądnie i obie przechodzą code review.

Razem tworzą **darmowe przewijanie wyniku do skutku**.

## Why it fails

Zwrot surowca przy przerwaniu znosi koszt nieudanej próby. Skoro próba nic nie kosztuje, gracz
powtarza ją aż do najlepszego wyniku. **Cała ekonomia jest wtedy liczona na wyniku minimalnym,
a gra realnie chodzi na maksymalnym.**

Wzmacniacz, który zamienia to z teoretycznego problemu w pewny exploit: **wynik jest pokazywany
PRZED zatwierdzeniem** (osobna faza "obejrzyj wynik" -> "kliknij, żeby odebrać"). Wtedy powtarzanie
nie jest ślepe, tylko **z podglądem wyniku** - gracz nie musi nawet zgadywać, kiedy się opłaca
przerwać. Widzi "SŁABY", wciska ESC, dostaje surowiec z powrotem.

Drugi, cichszy wariant: gdy minigra przerabia PARTIĘ (kolejka kilku sztuk), pole z wynikiem trzyma
wynik POPRZEDNIEJ sztuki. Naiwna naprawa ("przy przerwaniu zatwierdź to, co jest") zatwierdza wtedy
wynik poprzedniej sztuki - czyli zamienia jeden exploit na drugi.

## Symptoms

- W kodzie: `StartProcessing()` / `RemoveFromStorage()` przy wejściu w minigrę, a `AddToStorage()`
  w metodzie `Abort()` / `Cancel()` / `OnDisable()`.
- Silnik ma poprawną intencję (`CancelProcessing` z komentarzem "surowiec NIE jest zwracany"),
  ale **każdy interfejs minigry dokłada surowiec z powrotem ręcznie**, obchodząc własną specyfikację.
  Tego nie widać, patrząc tylko na silnik.
- Balans "nie działa": gracze produkują wyraźnie więcej, niż wynika z symulacji.
- Istnieje faza "pokaż wynik" przed fazą "zatwierdź".

## Correct approach

Wyznacz **jawny punkt bez odwrotu** i powiąż go z widocznym w grze zdarzeniem (np. "wciśnięcie
czerwonego guzika = ostrze rusza"). Przed nim: zwrot surowca jest uczciwy (nic się nie stało).
Po nim: przerwanie = **poddanie się** - liczy się to, co gracz zdążył (nierozegrane fazy = 0 punktów),
surowiec zużyty.

Trzy rzeczy, które trzeba zrobić dobrze:

1. **Nie karz utratą całego surowca** - zatwierdź wynik cząstkowy. Sprawdź jednak, czy "poddanie się"
   nie daje CZEGOŚ ZA DARMO: jeśli najgorszy wynik jest i tak osiągalny przez samo nicnierobienie
   w minigrze, to "przerwanie = najgorszy wynik" nie daje graczowi nic nowego - i wtedy commit jest
   bezpieczny. Jeśli najgorszy wynik wymaga wysiłku, commit cząstkowy staje się skrótem.
2. **Flaga "punktacja ruszyła"**, a nie enum faz. Enumy faz mają dziury (stan `None` podczas animacji
   kamery/klapy), przez które przecieka zła gałąź.
3. **Flaga "wynik jest świeży"** przy przetwarzaniu partii - inaczej przerwanie w połowie drugiej
   sztuki zatwierdzi wynik pierwszej.

Sam wskaźnik/pasek w trakcie minigry też zdradza wynik, więc faza punktowania liczy się już jako
"po punkcie bez odwrotu" - nie tylko ekran z wynikiem.

## Case study (Timber Tycoon, 2026-07-14)

Pięć minigier (piła taśmowa, peleciarka, wytwórnia nawozu, rębak, stolarnia) miało ten sam wzorzec.
Najgorzej w stolarni: jakość mnoży wypłatę (x1.25 / x1.5 / x2.0), więc przewijanie do Majstersztyku
było najbardziej opłacalnym działaniem w całej grze. Cała ekonomia (łącznie z wcześniejszym pełnym
rebalansem poziomów 1-13) była strojona na wynikach minimalnych. **Każdy pomiar balansu zrobiony
przed naprawą tego był bezwartościowy.**
