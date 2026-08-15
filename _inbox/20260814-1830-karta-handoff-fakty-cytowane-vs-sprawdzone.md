---
title: Karta handoffu mówiąca „nie sprawdzaj tego drugi raz" bez rozdzielenia faktów sprawdzonych od cytowanych
type: anti-pattern
status: draft
confidence: high
verified: ''
date: 2026-08-14
project: Another Quest
tags: [handoff, kontekst-miedzy-sesjami, dokumentacja, agent-workflow, stale-facts]
applies_to: [claude-code, handoff-cards, multi-session-workflow]
source: 'sesja errat B8, 2026-08-14 — karta _Handoff/KARTA_B8_ERRATY_2026-08-14.md'
suggested-category: workflow/anti-patterns
---

# Karta handoffu mówiąca „nie sprawdzaj tego drugi raz" bez rozdzielenia faktów sprawdzonych od cytowanych

## The trap

Karta zadania przekazywana między sesjami agenta (po `/clear`, do nocnego joba) niesie sekcję
w rodzaju **„Stan faktyczny na dziś — czego NIE musisz odkrywać. Sprawdzone w tej sesji.
Nie rób tego rekonesansu drugi raz."**

Intencja jest słuszna i oszczędza realny kontekst: świeża sesja nie musi odkopywać, która litera erraty
jest wolna, jak wygląda tabela rejestru, gdzie stoi parser. Pułapka polega na tym, że **ta sama karta
zawiera obok faktów RZECZYWIŚCIE sprawdzonych także fakty CYTOWANE z innych dokumentów** — i nie
odróżnia jednych od drugich. Świeża sesja, posłuszna instrukcji „nie sprawdzaj drugi raz", bierze
cytat za pomiar.

## Why it fails

**Fakt sprawdzony i fakt zacytowany starzeją się w zupełnie innym tempie.**

- Fakt sprawdzony (`grep` po repo, odczyt pliku, uruchomienie bramki) jest prawdziwy **w chwili pomiaru**
  i psuje się tylko wtedy, gdy ktoś ruszy to konkretne miejsce.
- Fakt zacytowany z innego dokumentu (rejestru pytań otwartych, listy blokerów, notatki z bramki
  decyzyjnej) jest prawdziwy **w chwili powstania TAMTEGO dokumentu** — czyli mógł być nieaktualny
  już w momencie pisania karty.

W obserwowanym przypadku karta powstała po południu i cytowała pozycję z rejestru pytań otwartych
(„klucz `CharacterId` nie występuje w repo ani razu, blokuje LOCK"). Ta pozycja została **zamknięta
decyzją tego samego dnia, kilka godzin wcześniej, osobnym commitem**. Karta była więc nieaktualna
**w dniu narodzin**, nie po tygodniu. Świeża sesja miała napisać do trwałego rejestru zdanie
„tego klucza nie ma w repo" — czyli **wpisać nieprawdę do dokumentu o najwyższej wadze w hierarchii
źródeł prawdy**.

Wzmacniacz: karta ma naturalny autorytet. Została napisana przez sesję, która **właśnie** siedziała
w temacie, jest świeża i mówi wprost „sprawdzone". Świeża sesja nie ma powodu jej nie wierzyć — a
instrukcja „nie rób tego rekonesansu drugi raz" jawnie zniechęca do weryfikacji.

## Symptoms

- Karta ma sekcję „stan faktyczny / czego nie musisz odkrywać" **i** sekcję z pytaniami otwartymi,
  blokerami albo długami — a fakty z tej drugiej są napisane tym samym tonem pewności co z pierwszej.
- W karcie stoi fakt w formie negatywnej: „X nie istnieje", „nikt tego nie rozstrzygnął",
  „brak zgody na Y". **Negatywy starzeją się najszybciej** — wystarczy jedna decyzja, żeby przestały
  być prawdą, a nikt nie wraca do cudzej karty, żeby ją poprawić.
- Karta cytuje inny dokument bez daty ostatniej zmiany tamtego dokumentu.
- Fakt w karcie dotyczy **decyzji człowieka** (zgoda, werdykt, wybór), a nie stanu plików. Decyzje
  zmieniają się między sesjami z definicji — to jest ich rola.

## Correct approach

**1. Rozdziel w karcie dwa rodzaje faktów, jawnie.** Sekcja „sprawdzone tu i teraz" dostaje przy każdej
pozycji **jak** sprawdzono (`grep`, odczyt, wynik bramki). Fakty cytowane idą do osobnej sekcji
z etykietą **„cytowane z <dokument> — zweryfikuj przed użyciem"**.

**2. Każdy fakt negatywny weryfikuj zawsze, niezależnie od tego, co mówi karta.** „X nie istnieje"
kosztuje jeden `grep`. To najtańsza weryfikacja w całym procesie i chroni przed najdroższym błędem —
wpisaniem nieprawdy do dokumentu trwałego.

**3. Fakty dotyczące decyzji człowieka mają w karcie termin ważności, nie datę powstania.**
Zamiast „BRAK ZGODY" pisz „BRAK ZGODY **na stan z <data> <godzina>**; zapytaj, nie zakładaj".

**4. Waga weryfikacji rośnie z trwałością zapisu.** Fakt, który wyląduje w scratchu — wierz karcie.
Fakt, który wyląduje w rejestrze, kanonie, migracji albo save'ie — sprawdź, choćby karta trzy razy
mówiła, że nie trzeba. Koszt sprawdzenia jest stały, koszt pomyłki rośnie.

**5. Karta powinna nazywać własne ryzyko starzenia.** Jedno zdanie: „Fakty z sekcji <X> są cytatami
z <data>; jeśli między nią a twoją sesją padły decyzje, one wygrywają."

Ta lekcja jest niezależna od projektu i silnika — dotyczy każdego procesu, w którym jedna sesja agenta
pisze instrukcję dla następnej.
