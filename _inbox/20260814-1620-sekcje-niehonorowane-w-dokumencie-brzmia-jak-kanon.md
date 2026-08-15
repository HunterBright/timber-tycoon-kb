---
title: Częściowo honorowany dokument produkuje fałszywe cytaty
type: anti-pattern
status: draft
confidence: high
verified: '2026-08-14'
date: 2026-08-14
project: Another Quest
tags: [dokumentacja, hierarchia-zrodel, kanon, agenci, cytowanie]
applies_to: [dokumentacja-projektowa, praca-z-llm, wielosesyjne-projekty]
source: 'Batch B8 (QUEST/GUILD/HUB) — sześć niezależnych przeglądów wykryło ten sam błąd cytatu'
suggested-category: workflow/anti-patterns
---

# Częściowo honorowany dokument produkuje fałszywe cytaty

## Pułapka

Masz stary dokument (GDD v1, stara specyfikacja, notatki z kick-offu), który częściowo się zdezaktualizował.
Zamiast go archiwizować, zapisujesz regułę: **„honorujemy z niego wyłącznie sekcję 2"**. Wygląda to na tani
i precyzyjny kompromis — sekcja 2 jest aktualna, reszta jest historią, wszyscy wiedzą.

Nikt nie wie. **Wszystkie sekcje wyglądają identycznie** — ta sama czcionka, ten sam plik, ten sam autorytet
w nazwie. Reguła żyje w innym pliku niż tekst, którego dotyczy.

## Dlaczego to zawodzi

Czytający — człowiek albo agent — otwiera dokument, znajduje zdanie, które odpowiada na jego pytanie,
i cytuje je jako obowiązujące. Żeby tego nie zrobić, musiałby przy **każdym** cytacie policzyć numery linii
i porównać z granicami sekcji. Nikt tego nie robi, bo koszt jest ponoszony za każdym razem, a nagroda
jest niewidoczna.

Efekt jest gorszy niż zwykły błąd: **materiał historyczny wraca do obiegu z pieczątką kanonu.** W projekcie AQ
jedna linia takiego dokumentu (l. 175) niosła kilkadziesiąt decyzji z różnych epok, w tym część już
**odwróconą erratami**. Szkic zbudował na niej twardy warunek walidacji i nazwał go kanonem, a gdy krytyk
zgłosił zastrzeżenie — **szkic odrzucił trafną uwagę, „sprawdzając" numer sekcji błędnie.**

Drugi mechanizm: reguła wygląda na łatwą do sprawdzenia, więc weryfikator jej nie sprawdza. Sześciu
niezależnych agentów wykryło ten sam błąd dopiero wtedy, gdy dostali polecenie **otworzyć plik i policzyć
linie nagłówków**, a nie „sprawdzić, czy cytat jest wierny".

## Objawy

- Dokument ma w metryce zapis w rodzaju „honorowana wyłącznie sekcja N" albo „STALE, ale sekcja X obowiązuje".
- Cytaty z tego dokumentu w nowych dokumentach nie podają numeru linii, tylko numer sekcji — **z pamięci**.
- Ta sama treść bywa cytowana raz jako kanon, raz jako materiał historyczny, w dwóch dokumentach tego samego dnia.
- Zdanie „to stoi w sekcji honorowanej" pojawia się **jako odparcie zarzutu**, bez otwarcia pliku.

## Poprawne podejście

**Fizycznie rozdziel to, co obowiązuje, od tego, co nie.** Granica statusu ma być granicą pliku, nie akapitu:

1. Przenieś sekcję honorowaną do osobnego dokumentu z własną nazwą (w AQ: `00_MATRYCA_ZALEZNOSCI.md`
   przejęła funkcję sekcji 2 i to zadziałało — nikt nigdy nie pomylił jej statusu).
2. Resztę przenieś do `archive/` z nagłówkiem mówiącym wprost: „to jest zapis myślenia, nie zapis decyzji".
3. Jeśli nie możesz rozdzielić od razu — **wstaw ostrzeżenie do samego pliku, na początku każdej
   niehonorowanej sekcji**, nie tylko do metryki. Koszt czytającego musi spaść do zera.

**Zanim rozdzielisz, zmierz szkodę:** wygrepuj każdą frazę, którą projekt cytuje z tego dokumentu,
i sprawdź, czy ma **drugie źródło**. W AQ okazało się, że większość treści miała drugą, legalną podstawę
w innym dokumencie — więc decyzje się broniły, a błędne były wyłącznie odsyłacze. Cztery zapisy nie miały
żadnej podstawy i przez pół roku funkcjonowały jako kanon.

## Sygnał, że jesteś w tej pułapce dziś

Zadaj sobie pytanie: *„gdyby ktoś zacytował mi zdanie z tego pliku, czy umiałbym bez otwierania go powiedzieć,
czy ono obowiązuje?"* Jeśli nie — twoi współpracownicy i agenci też nie umieją, tylko tego nie zgłaszają.
