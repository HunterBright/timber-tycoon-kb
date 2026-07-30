---
type: pattern
project: Kerf - Sawmill Tycoon
suggested-category: workflow/tooling
tags: [dependency-management, closed-source, changelog, upgrade-decision, mcp]
date: 2026-07-28
status: draft
---

# Wtyczka bez changelogu: różnica ciągów znaków w DLL zamiast zgadywania

## When to use
Trzeba zdecydować o aktualizacji zamkniętej wtyczki (jeden obfuskowany DLL), a producent
nie publikuje listy zmian: brak release notes na GitHubie, brak strony changelog w docs,
komunikaty commitów to tylko "Release vX.Y.Z".

## Steps
1. Płytki klon repo wtyczki na branch wydań, odczyt `package.json` = wersja HEAD.
2. Wyciągnąć ciągi ASCII i UTF-16LE (min. 5-6 znaków) ze starego i nowego DLL, zrobić
   różnicę zbiorów (dodane / usunięte).
3. Odfiltrować szum (System.*, Newtonsoft.*, losowe symbole obfuskatora), zostawić
   czytelne nazwy typów, pól i metod.
4. Dla podejrzanych funkcji sprawdzić OBECNOŚĆ w obu wersjach osobno - numery stanów
   maszyn asynchronicznych (`<Metoda>d__42`) przesuwają się przy każdej kompilacji
   i generują fałszywe "nowości".

## Why this works
Nazwy typów i pól przetrwają obfuskację na tyle, że widać kierunek zmian. W naszym
przypadku różnica dała jednoznaczny obraz: doszły wyłącznie nowe identyfikatory modeli AI
plus warstwa rozliczeń (odcisk sprzętowy, powód odmowy darmowych kredytów). Ścieżki
uruchamiania skryptów i kompilacji miały identyczne ciągi, czyli znany błąd nie został
naprawiony i aktualizacja nic by nie dała.

## Trade-offs
- To poszlaki, nie dokumentacja. Nie zobaczysz zmian czysto behawioralnych.
- Nie działa na kodzie mocno obfuskowanym łącznie z literałami.

## Variants
Ta sama metoda odpowiada na pytanie "czy warto podbić" dla dowolnego zamkniętego SDK.

## Reguła, która z tego wypadła
**Most (wtyczka po stronie narzędzia) i serwer (proces po stronie agenta) muszą być
dobraną parą.** Wtyczka miała datę 23.02, najnowszy serwer 25.02 - dobrana para.
Nowa wtyczka była z 29.06, ale serwer stał w miejscu od lutego. Aktualizacja samej wtyczki
zrobiłaby rozjazd 4 miesięcy, nie ulepszenie. Przed podbiciem jednej strony mostu
zawsze sprawdzić datę ostatniego wydania drugiej strony.

## Sygnał ostrzegawczy przy okazji
Serwer zamrożony od miesięcy + nota w README o przejęciu produktu = narzędzie wygasa.
Wtedy pytanie brzmi nie "aktualizować?", tylko "czym to zastąpić?".
