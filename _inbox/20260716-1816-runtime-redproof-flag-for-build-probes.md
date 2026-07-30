---
type: pattern
project: Timber Tycoon
suggested-category: workflow/patterns
tags: [testing, build-probe, falsifiability, red-proof, command-line]
date: 2026-07-16
status: draft
---

# Flaga runtime "-redproof" zamiast drugiego builda do czerwonego dowodu sondy

## Kontekst
Twarda reguła projektu: każdy nowy check sondy buildowej musi mieć UDOWODNIONY tryb porażki.
Dotychczasowy sposób: zbuduj grę z bugiem (przed fixem), uruchom test, zobacz FAIL, potem
zbuduj z fixem i zobacz PASS. Koszt: DWA pełne buildy (kilkanaście minut każdy) i dowód
nie jest powtarzalny po scaleniu fixa.

## Wzorzec
Dodaj statyczną flagę czytaną z argumentów linii komend (np. `Day6RedProof.SpotFixDisabled =
HasArg("-spotredproof")`) i zbramkuj nią WSZYSTKIE miejsca naprawy (po jednej linijce na
miejsce). Wtedy JEDEN build dowodzi obu stron, powtarzalnie na zawsze:
- `gra.exe -test -redproof` -> exit 1 (bug odtworzony, check gryzie),
- `gra.exe -test` -> exit 0 (naprawa działa).

## Zasady bezpieczeństwa
- Flaga TYLKO wyłącza naprawy, nigdy nie zmienia logiki gry w inny sposób.
- Domyślnie false (brak argumentu = pełna naprawa) - zwykły gracz nigdy jej nie poda.
- Komentarz przy każdej bramce: "czerwony dowod sondy X" - czytelnik wie, czemu if istnieje.
- Uzupełnia (nie zastępuje) wbudowany samotest detektora (podstawiony fałszywy obiekt musi
  być wykryty) - flaga dowodzi scenariusza end-to-end, samotest dowodzi detektora.

**Why:** falsyfikowalność sondy bez podwójnych buildów; dowód porażki zostaje w repo i można
go powtórzyć przy każdej regresji.
**How to apply:** gdy naprawiasz bug i dopisujesz check do sondy, od razu zbramkuj naprawę
flagą -redproof i dodaj do bramki builda parę uruchomień (z flagą = oczekiwany exit 1).
