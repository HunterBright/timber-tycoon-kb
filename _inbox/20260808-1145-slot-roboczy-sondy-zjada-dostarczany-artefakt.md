---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [testy, sonda, zapisy-gry, artefakty, kolizja-zasobow, unity]
date: 2026-08-08
status: draft
---

# Slot roboczy sondy kasuje artefakt, który właśnie dostarczyłeś

## Objaw

Przygotowany dla użytkownika zapis gry (slot 2) zniknął z dysku między jednym poleceniem
a drugim. Zapis powstał, został skontrolowany przez wczytanie, wynik zielony - a po
uruchomieniu rutynowej sondy siatek plik po prostu przestał istnieć.

## Przyczyna

Slot 2 był slotem ROBOCZYM sond automatycznych. Dwie z nich (`MeshReadabilityProbe`,
`SaveLoadTreeProbe`) sprzątają po sobie, kasując `save_2.json` na końcu przebiegu.
Sprzątanie jest poprawne - błędem było wybranie tego samego zasobu na trwały artefakt.

Wybrałem slot 2 z przyzwyczajenia po tym, jak slot 9 okazał się poza zakresem
(gra ma sloty 0-2). Nie sprawdziłem, czy ktoś już go używa.

## Lekcja

**Zanim zapiszesz trwały artefakt w numerowany zasób współdzielony (slot zapisu, port,
nazwa pliku tymczasowego, tabela testowa), przeszukaj repo pod kątem tego, kto ten zasób
KASUJE.** Nie kto go zapisuje - kto go kasuje. Zapis konkurencyjny objawia się od razu,
skasowanie objawia się dopiero przy następnym przebiegu czegoś niezwiązanego.

Jedno wyszukanie po `Delete` w kodzie testowym rozstrzyga sprawę w minutę.

## Drugi wniosek, ważniejszy

Sprawdzenie „czy plik istnieje i ma dobrą treść" ma wartość tylko wtedy, gdy zrobisz je
PO wszystkich krokach, które zamierzasz wykonać - także po tych „niezwiązanych"
(build, sonda, testy). Kontrola przeprowadzona w środku sekwencji dowodzi stanu w środku
sekwencji, a nie stanu, który dostanie użytkownik.

## Powiązane

Ta sama rodzina co lekcja o starym raporcie uznanym za dowód: artefakt na dysku nie jest
dowodem sam z siebie, dopóki nie wiesz, kiedy powstał i co go od tamtej pory dotykało.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260720-0920-red-proof-musi-uzbroic-wszystkie-checki|Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony]] - wspolne: sonda, testy
- [[gate-must-have-provable-failure-mode|Bramka bez udowodnionego trybu porazki niczego nie pilnuje]] - wspolne: sonda, testy
<!-- /POWIAZANE:auto -->
