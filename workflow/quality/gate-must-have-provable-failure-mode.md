---
title: Bramka bez udowodnionego trybu porazki niczego nie pilnuje
type: pattern
status: verified
confidence: high
verified: '2026-07-30'
date: '2026-07-30'
project: Kerf - Sawmill Tycoon
tags:
- qa
- walidacja
- sonda
- testy
- red-proof
- metodologia
applies_to:
- unity-projects
- blender-pipelines
- dowolny-pipeline-walidacyjny
source: konsolidacja 10 odnosnikow do nieistniejacego wpisu podczas migracji do Obsidiana
---

# Bramka bez udowodnionego trybu porazki niczego nie pilnuje

Ten wpis powstal, bo w bazie wiedzy bylo **dziesiec odnosnikow** do tego pojecia
pod szescioma roznymi nazwami (`probe-must-be-able-to-fail`,
`probe-check-must-have-provable-failure-mode`, `sonda-musi-umiec-zawiesc`,
`probe-checks-need-a-proven-failure-mode`,
`probe-checks-must-name-the-culprit-not-just-fail`) i ani jednej notatki.
To jest najczesciej przywolywana zasada w calym projekcie.

## Kiedy stosowac

Zawsze, gdy dopisujesz sprawdzenie do sondy, test, walidator albo bramke buildowa.
Bez wyjatkow.

## Zasada

Sprawdzenie, ktorego **nie da sie celowo zlamac**, nie jest sprawdzeniem.
Jest ozdoba, ktora zapala zielone swiatlo niezaleznie od stanu projektu.

Kazde nowe sprawdzenie musi miec dzwignie, ktora je celowo psuje (red-proof),
i trzeba ja **uruchomic raz** przy dodawaniu checku. Jesli po zlamaniu warunku
bramka nadal przechodzi - bramka jest zepsuta, a nie kod.

## Kroki

1. Napisz sprawdzenie.
2. Napisz przelacznik, ktory celowo lamie warunek (flaga wiersza polecen,
   podmieniony asset, wymuszona zla wartosc).
3. Uruchom z przelacznikiem. Bramka **musi** oblac i **musi nazwac winowajce**.
4. Uruchom bez przelacznika. Bramka musi przejsc.
5. Dopiero teraz sprawdzenie ma prawo istniec w zestawie.

Punkt 3 ma dwie czesci i obie sa obowiazkowe. Bramka, ktora oblewa komunikatem
"cos jest nie tak", kosztuje potem godzine szukania. Ma podac nazwe obiektu,
sciezke i zmierzona wartosc.

## Dlaczego to dziala

Test sprawdza dwie rzeczy naraz: stan projektu i samego siebie. Bez trybu porazki
weryfikujesz tylko to, ze kod testu sie wykonal - a to zawsze prawda.
Zielone swiatlo z takiego testu jest gorsze niz brak testu, bo daje falszywa pewnosc.

## Odmiany tego samego bledu

- **Walidator spelniony przez konstrukcje** - sprawdza warunek, ktory generator
  wymusza z definicji (np. mierzy wysokosc, ktora sam ustawil). Zawsze zielony.
- **Bramka mierzaca wlasna definicje** - liczy wynik z tych samych liczb,
  z ktorych powstal obiekt.
- **Pula jednoelementowa udajaca pelne pokrycie** - test przechodzi po jednym
  elemencie i raportuje "wszystkie OK".
- **Bramka ponad sufitem** - prog ustawiony tak wysoko, ze nic go nie przekroczy.
- **Test wzgledny slepy na blad wspolny** - porownuje modele miedzy soba,
  wiec nie widzi bledu, ktory dotyka wszystkich tak samo.

## Dowod z tego projektu

- `Assets/Project/Scripts/Testing/MeshReadabilityProbe.cs` ma red-proofy wbudowane
  w kod: `-polishredproof`, `wheelRedProof`, `VFX/redproof`, `EK/samotest`,
  `Stopy/samotest`, `UI/samotest detektora`.
- `_BlenderOutputs/NPCCar01/selftest_validators.py` - kazdy walidator z `car_spec.py`
  ma tam dzwignie, ktora go celowo lamie. Komentarz w pliku mowi wprost:
  "check, ktorego NIE DA SIE zepsuc, nie pilnuje niczego".
- `CLAUDE.md` projektu, sekcja "Bramka nr 3: sonda rosnie" - kazda naprawa z rodziny
  bledow buildowych dopisuje do sondy sprawdzenie, ktore by ja wylapalo.
- Sonda ma tez straznikow sekcji: po kazdej korutynie `if (!xSectionDone) Fail(...)`,
  zeby wyjatek w srodku nie zapalil zielonego.

## Powiazane

- [[build-is-the-only-truth-editor-lies]]
- [[20260714-1245-test-bez-trybu-porazki]]
- [[20260720-0920-red-proof-musi-uzbroic-wszystkie-checki]]
- [[20260727-1422-bramka-musi-umiec-zaliczyc-nie-tylko-oblac]]
- [[20260716-1816-runtime-redproof-flag-for-build-probes]]
- [[20260720-1306-walidator-spelniony-przez-konstrukcje]]
- [[20260720-1308-pula-jednoelementowa-udaje-pelne-pokrycie]]
- [[20260728-1500-bramka-ponad-sufitem]]
- [[20260722-1652-relative-only-test-blind-to-common-mode-error]]

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260720-0920-red-proof-musi-uzbroic-wszystkie-checki|Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony]] - wspolne: red-proof, testy, sonda
- [[20260731-0025-sedzia-na-artefaktach-lapie-bledy-wlasnego-narzedzia|Sedzia oceniajacy artefakty lapie bledy w narzedziu, ktore te artefakty produkuje]] - wspolne: metodologia, walidacja, qa
- [[build-is-the-only-truth-editor-lies|Edytora nie da sie oszukac, zeby udawal build]] - wspolne: metodologia, sonda, qa
- [[SEDZIA|Sędzia jakości - na czym go zbudowaliśmy]] - wspolne: walidacja, qa
- [[20260731-2200-slepa-dzwignia-debugger-bramek|20260731-2200-slepa-dzwignia-debugger-bramek]] - wspolne: sonda, qa
<!-- /POWIAZANE:auto -->
