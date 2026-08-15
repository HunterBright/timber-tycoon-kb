---
title: Wyczerpanie przestrzeni nazw po cichu łamie reguły dopasowujące po prefiksie
type: lesson
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags:
- bramki
- ciche-awarie
- konwencje-nazw
- narzedzia
- testowanie
applies_to: []
source: 'sesja dzienna Another Quest, nadanie pierwszych dwuliterowych errat; finding NARZ-03'
severity: high
suggested-category: workflow/lessons
time_lost: '~20 min od objawu do naprawy'
---

# Wyczerpanie przestrzeni nazw po cichu łamie reguły dopasowujące po prefiksie

## Problem

Rejestr poprawek do dokumentów używał jednoliterowych identyfikatorów: `a`, `b`, … `z`.
Kilka wpisów było rozbitych na części — `k1`, `k2`, `k3` — i miało dziedziczyć konfigurację po `k`.
Zrobiono to najprostszym możliwym sposobem:

```python
def klucz_wyjatku(litera):
    return litera[0] if litera and litera[0].isalpha() else litera
```

Przez wiele tygodni działało bezbłędnie, bo **wszystkie identyfikatory były jednoliterowe albo
litera+cyfra**. Potem alfabet się wyczerpał i pojawiły się `aa`, `ab`.

Od tego momentu `aa` dziedziczyło konfigurację po `a`, a `ab` po `b`. Konkretnie: przejmowały
wyjątek „ta pozycja nie ma terminu, nigdy nie zgłaszaj jej jako zaległej" — więc **nowe wpisy
z realnym terminem nigdy nie zapaliłyby się na czerwono**.

**Bramka świeciła na zielono dokładnie tam, gdzie miała ostrzegać.** Nikt by się nie dowiedział,
bo zielony wynik nie budzi podejrzeń — sprawdza się czerwone, nie zielone.

Wykryte przypadkiem: raport bramki pokazał przy nowym wpisie uzasadnienie wyjątku, które ewidentnie
dotyczyło czegoś innego („treść wpleciona w moduły 7 i 8" przy wpisie adresującym moduł 20).
Gdyby to uzasadnienie nie było drukowane, błąd żyłby dalej.

## Root cause

Reguła „weź pierwszy znak" **kodowała założenie o rozmiarze przestrzeni nazw**, nie wypowiadając go.
Założenie brzmiało: *„identyfikator to jedna litera, ewentualnie z cyfrą"*. Było prawdziwe w dniu
napisania i przestało być prawdziwe, gdy przestrzeń się wyczerpała.

Dwa czynniki, które zrobiły z tego cichą awarię, a nie głośną:

1. **Dopasowanie po prefiksie nigdy nie zawodzi jawnie.** Zawsze coś dopasuje — po prostu nie to.
2. **Skutkiem było ROZLUŹNIENIE kontroli, nie zaostrzenie.** Gdyby błąd powodował fałszywe alarmy,
   ktoś zgłosiłby to pierwszego dnia. Fałszywy spokój nikogo nie uwiera.

## Solution

**Zawęź regułę do tego, co faktycznie miała łapać, i zapisz to w kodzie jako intencję:**

```python
def klucz_wyjatku(litera):
    # dziedziczenie WYŁĄCZNIE po sufiksie liczbowym: k1 -> k.
    # 'aa' to kontynuacja alfabetu, czyli wpis CAŁKOWICIE NIEZALEŻNY.
    if len(litera) > 1 and litera[0].isalpha() and litera[1:].isdigit():
        return litera[0]
    return litera
```

**I dołóż test, który pada bez tej poprawki.** W testowym zestawie danych postaw wpis `aa`
z terminem w przeszłości **obok** wpisu `a` oznaczonego jako zwolniony z terminów, i wymagaj,
żeby `aa` został zgłoszony.

**Udowodnij, że test mierzy.** Przywróć starą implementację (choćby podmieniając funkcję w pamięci
na czas jednego uruchomienia) i sprawdź, że test pada dokładnie na tym przypadku. Test dopisany
do zielonego zestawu bez tego kroku jest tylko deklaracją, że coś się przetestowało.

## What didn't work

- **Zmiana konwencji zamiast naprawy kodu.** Pierwsza myśl: „to może nie używać dwuliterowych".
  To zostawia minę — następna osoba, która ich potrzebuje, wpadnie w to samo, a przyczyna
  zostanie nietknięta.
- **Poleganie na zielonej bramce.** Bramka była zielona przez cały czas trwania błędu i byłaby dalej.
  Zielony wynik jest dowodem tylko wtedy, gdy wiadomo, że czerwony jest osiągalny.

## Transferability

Ten sam kształt występuje wszędzie, gdzie identyfikator jest **rozbierany na części zamiast
porównywany w całości**:

- reguły uprawnień dopasowujące ścieżkę po prefiksie (`/api/user` łapie też `/api/users-admin`),
- routing po pierwszym segmencie,
- feature flagi z dziedziczeniem po kropce (`app.foo` vs `app.foobar`),
- konfiguracje środowisk (`prod` dopasowuje `production-readonly`),
- nazwy gałęzi, tagów, kolejek, tematów.

Dwa pytania kontrolne przy każdej takiej regule:

1. **Jakie założenie o rozmiarze przestrzeni nazw tu zakodowałem** i co się stanie, gdy przestrzeń
   urośnie albo się wyczerpie?
2. **Czy błąd tej reguły objawi się rozluźnieniem, czy zaostrzeniem kontroli?** Jeśli rozluźnieniem —
   potrzebuje testu od pierwszego dnia, bo nikt go nie zgłosi.

Ogólniej: **wyczerpanie przestrzeni nazw to zdarzenie, które ujawnia wszystkie ukryte założenia
o jej kształcie naraz.** Warto sprawdzić resztę systemu przy okazji.

## Related

- [[test-hooka-musi-isc-kanalem-poza-permissions]] — ta sama rodzina: ochrona, której awarii nie
  da się odróżnić od poprawnego działania
- [[wyluskaj-niezmiennik-zanim-obalisz-decyzje]] — tam też chodzi o niewypowiedziane założenie
  zaszyte w starej decyzji
