---
title: Przedawnienie wpisu jest funkcja jego TYPU, nie uplywu czasu
type: pattern
status: draft
confidence: medium
verified: ''
date: 2026-08-05
project: GameDevOS
tags: [baza-wiedzy, metadane, przedawnienie, automatyzacja, agent]
applies_to: [obsidian, knowledge-base, claude-code]
source: 'https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf oraz https://www.glukhov.org/knowledge-management/knowledge-systems-architectures/compiled-knowledge/llm-wiki-maintenance-knowledge-drift/'
suggested-category: workflow/quality
---

# Przedawnienie wpisu jest funkcja jego TYPU, nie uplywu czasu

## Kiedy stosowac

Gdy baza wiedzy ma wiecej niz okolo dwustu wpisow i czyta ja agent, a nie tylko
czlowiek. Wtedy przestaje wystarczac zasada „stare wpisy sa podejrzane": jeden
globalny prog czasu **jednoczesnie krzyczy na wpisy, ktore sa wieczne, i milczy
o tych, ktore zgnily po miesiacu**.

Objaw, po ktorym poznajesz, ze to Twoj przypadek: agent podaje sprzeczne
odpowiedzi na to samo pytanie, zaleznie od tego, na ktora notatke trafil.

## Kroki

1. Nadaj kazdemu wpisowi pole `type`, jesli jeszcze go nie ma.
2. Zbuduj **tabele interwalow przegladu wedlug typu**, a nie jeden prog dla
   wszystkiego. Punkt wyjscia, ktory mozna wziac wprost:

   | Rodzaj tresci | Jak dlugo jest wazna |
   |---|---|
   | ceny i dostepnosc uslugi | 7 do 30 dni |
   | wersje narzedzi | 30 do 90 dni |
   | porownania narzedzi | 30 do 180 dni |
   | glosariusz, definicje | 3 do 12 miesiecy |
   | zasady architektoniczne | 6 do 18 miesiecy |

3. Zapisz w naglowku wpisu date, po ktorej wpis wymaga przegladu
   (`stale_after` albo `review_after`), wyliczona z typu, a nie wpisana z reki.
4. Osobno zapisz **odcisk zrodla** (hash tresci albo date zmiany), na ktorym wpis
   sie opieral. Przy nastepnym przegladzie porownanie dwoch napisow odpowiada
   na pytanie „czy zrodlo w ogole sie ruszylo".
5. Detektor przedawnienia to teraz porownanie dat i napisow. **Zero wywolan
   modelu jezykowego.**

## Dlaczego to dziala

Bo przedawnienie nie jest wlasciwoscia czasu, tylko wlasciwoscia **tempa zmian
dziedziny, ktorej wpis dotyczy**. Notatka o cenniku uslugi w chmurze i notatka
o tym, jak myslec o architekturze, starzeja sie w zupelnie innym tempie, wiec
jeden prog musi sie mylic przy jednej z nich.

Druga polowa dziala dlatego, ze **wiekszosc „sprzecznosci" w bazie technicznej
wcale nia nie jest**: to dwa twierdzenia o roznych wersjach, roznych zakresach
albo z roznych dat. Krok klasyfikacji przed zgloszeniem odsiewa je zanim trafia
do raportu.

## Kompromisy

- Trzeba utrzymywac tabele typow. Za to jest to jedna tabela, a nie regula
  przy kazdym wpisie.
- Detektor **nie wie**, czy tresc jest prawdziwa. Wie tylko, ze minal termin
  przegladu. To jest celowe: automat, ktory sam „naprawia" fakty, potrafi
  rozniesc bledna wartosc po calej bazie w jednym przebiegu.
- Wymaga dyscypliny przy zapisywaniu zrodla. Wpis, ktory podaje **tytul**
  dokumentu zamiast **adresu**, jest nie do sprawdzenia przy nastepnym przegladzie.

## Warianty

**Gotowy kontrakt metadanych zamiast wlasnego.** Open Knowledge Format w wersji
0.2 (Google Cloud, migracja formatu 24.07.2026) definiuje dokladnie te pola:
`status` o wartosciach `draft` / `stable` / `deprecated`, `stale_after` z data,
`sources` z data ostatniej zmiany kazdego zrodla, `verified` z data i autorem
sprawdzenia oraz `generated` z informacja, kto wpis wytworzyl. Format jest
zwyklym katalogiem plikow markdown z naglowkiem YAML, wiec **nie wymaga zmiany
narzedzi**. Warto wziac podzbior, nie calosc.

**Uwaga o zrodle:** strona `okf.md` opisuje wersje 0.1 i **nie zawiera pol
`stale_after` ani `status`**. Pelny zestaw jest w repozytorium GoogleCloudPlatform.
Przy tym formacie idz za repozytorium, nie za strona.

## Czego to nie zastapi

Gdy fakt dotyczy **wlasnej maszyny**, pomiar bije kazdy wpis i kazde zrodlo.
Spor o to, na ktorej wersji edytora stoi projekt, rozstrzyga jeden plik
`ProjectVersion.txt`, a nie trzecia notatka w bazie.
