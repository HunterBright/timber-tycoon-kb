---
title: Nowy walidator uruchom najpierw na ZATWIERDZONYM artefakcie; jeśli tam zapala się na czerwono, błędny jest walidator, nie artefakt
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- validation
- regression-guard
- calibration
- golden-artifact
- code-review
applies_to:
- any-project-with-generated-assets
- validators
- regression-guards
source: ''
severity: high
time_lost: 0 - złapane przed implementacją, ale kosztowałoby dzień
suggested-category: process/lessons
---

# Nowy walidator uruchom najpierw na ZATWIERDZONYM artefakcie; jeśli tam zapala się na czerwono, błędny jest walidator, nie artefakt

## Problem

Przy dodawaniu trzeciego wariantu proceduralnie budowanego modelu (auto NPC) analiza
wskazała lukę: nic nie sprawdza, czy koło nie przebija blachy nadwozia. Propozycja
brzmiała naturalnie - dopisać kontrolę `linia_podłogi(x) - szczyt_opony >= margines`.

Przeliczenie tej kontroli na **zatwierdzonym i wysłanym** modelu pokazało, że zapaliłaby
się na nim na obu osiach: opona wchodzi w nadwozie o 3.3 mm z przodu i 29.0 mm z tyłu.

Autor propozycji sam napisał wniosek: "trzeba najpierw naprawić tylne nadkole sedana,
co złamie golden digest". Czyli walidator wymuszałby **zepsucie modelu, który przeszedł
akceptację**, tylko po to, żeby przejść własny próg.

## Root cause

Walidator kodował założenie zapożyczone z innej konstrukcji ("koło i nadwozie nie mogą
się przenikać"), a nie założenie tej konstrukcji. Ten model stoi na **przenikających się
bryłach zamkniętych**: wtopienie górnej części opony w zamknięte nadwozie JEST wnęką
koła. Nie ma tam żadnego wycięcia i nigdy nie miało być.

Mechanizm powstania błędu jest ogólny: analiza szuka "czego nie pilnuje żaden test",
znajduje prawdziwą lukę w pokryciu, i zakłada, że luka w pokryciu = niepilnowany defekt.
Tymczasem część luk to miejsca, w których **nie ma czego pilnować**, bo zachowanie jest
zamierzone. Bez uruchomienia na znanym-dobrym wzorcu nie da się tych dwóch przypadków
odróżnić.

## Solution

Do procedury dodawania walidatora wstaw krok kalibracji **przed** implementacją:

1. Policz wartość, którą walidator ma sprawdzać, na **wszystkich** już zatwierdzonych
   artefaktach. Nie na jednym - w tym projekcie kontrola zawierania kabiny przechodziła
   na sedanie z 20 cm zapasu, a na kombi z 1.6 mm.
2. Ustaw próg **z tego pomiaru**, nie z intuicji. Zapisz w komentarzu, ile zapasu
   zostaje najciaśniejszemu zatwierdzonemu artefaktowi i dlaczego to wystarcza.
3. Jeżeli walidator zapala się na zatwierdzonym artefakcie - **nie dodawaj go**.
   Albo mierzy nie to, co trzeba, albo koduje cudze założenia konstrukcyjne.
4. Dopisz kontrolę kalibracji do samotestu: "nietknięty wariant X przechodzi" musi być
   osobną, widoczną linią wyniku dla KAŻDEGO zatwierdzonego wariantu, nie tylko
   pierwszego.
5. Dopiero potem dopisz dźwignię, która walidator celowo łamie.

Punkt 4 jest tym, o którym najłatwiej zapomnieć: samotest z dźwigniami udowadnia, że
walidator UMIE zawieść, ale nie że **nie zawodzi bez powodu**.

## What didn't work

- **Przyjęcie tezy z notatki przekazującej sesję na wiarę.** Trzy tezy o tym, co złamie
  nowy wariant, spisane pod koniec poprzedniej sesji; po sprawdzeniu w kodzie **dwie
  były fałszywe**, a trzecia prawdziwa z niewłaściwego powodu. Notatki handoffowe
  zapisują hipotezy w trybie oznajmującym i po nocy czyta się je jak ustalenia.
- **Ocena luki w pokryciu jako defektu.** "Żaden test tego nie sprawdza" to zdanie
  o testach, nie o artefakcie.
- **Kalibracja na jednym wariancie.** Wszystkie dźwignie samotestu psuły wyłącznie
  pierwszy wariant, więc "walidatory są żywe" było twierdzeniem o jednym modelu
  z trzech. Wariant najbliższy progu bywa innym wariantem niż ten domyślny.

## Transferability

Dotyczy każdego projektu, w którym istnieją **zatwierdzone artefakty generowane kodem**
i dokłada się do nich kontrole: modele proceduralne, generatory konfiguracji, migracje
schematów, lintery na istniejącej bazie kodu, progi wydajnościowe.

Ogólna zasada: **walidator jest hipotezą o tym, co jest poprawne, i wymaga
falsyfikacji tak samo jak kod, który ma pilnować.** Jedynym dostępnym materiałem
dowodowym są artefakty, które już przeszły akceptację człowieka. Walidator sprzeczny
z nimi jest obalony, niezależnie od tego, jak sensownie brzmi jego uzasadnienie.

Odwrotność też jest praktyczna: jeśli nowy walidator przechodzi na wszystkich
zatwierdzonych artefaktach z sensownym zapasem, to zmierzony zapas jest gotowym,
uzasadnionym progiem - i najlepszym komentarzem, jaki można przy nim zostawić.

## Related
- [[20260720-1410-ortho-comparison-render-hides-occlusion]] - ten sam motyw: dowód, który wygląda
  wiarygodnie, a nie dowodzi tego, co obiecuje
- Zasada projektowa Timber Tycoon: "sonda musi umieć zawieść" - ta lekcja jest jej
  brakującą drugą połową: sonda musi też umieć NIE zawieść bez powodu
