---
title: 'Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)'
type: anti-pattern
status: draft
confidence: low
verified: ''
date: '2026-07-11'
project: Kerf - Sawmill Tycoon
tags:
- blender
- props
- z-fighting
- low-poly
- modeling
- qa
applies_to: []
source: ''
suggested-category: blender/lessons
---

# Anty-wzorzec: szczeliny powietrza jako ochrona przed z-fightingiem (lewitujące propy)

## Problem

Przy proceduralnym budowaniu sceny/modelu z wielu brył agent (lub skrypt) chroni się
przed z-fightingiem przez dodawanie ODSTĘPÓW między obiektami. Efekt: przedmioty,
które powinny na czymś leżeć lub wisieć, unoszą się w powietrzu o kilka-kilkanaście mm.
Na renderach z dystansu wygląda to znośnie, ale reżyser od razu wyłapał "wiszące albo
dziwnie leżące przedmioty" (pistolet 9 cm pod haczykiem, rączka piły 13 mm nad półką,
poziomy trzonek młotka 37 mm nad półką, belka ścianki 3 cm nad ziemią).

## Dlaczego to nie działa

Z-fighting powstaje TYLKO gdy dwie równoległe, nakładające się powierzchnie są
niemal współpłaszczyznowe (<~1 mm). Kontakt fizyczny obiektów nie wymaga odstępu -
wymaga PRZENIKANIA. Odstęp "dla bezpieczeństwa" nie usuwa żadnego realnego ryzyka,
a psuje wiarygodność sceny (grawitacja przestaje działać).

## Poprawny wzorzec

- Obiekt leżący na powierzchni: spód WPUŚĆ 1-2 mm w powierzchnię (płaszczyzny ścianek
  są wtedy >1 mm od siebie - zero z-fightingu, zero lewitacji).
- Elementy złączone (trzonek w głowicy, ostrze w rączce): przenikanie 10-20 mm.
- Obiekt wiszący na haku: geometria MUSI się nakładać z hakiem (hak przechodzi
  "przez" korpus), nie wisieć pod nim.
- Leżące podłużne przedmioty (młotek, pędzel) - oś pozioma tylko gdy obie podpory na
  tej samej wysokości; inaczej PRZECHYL, żeby drugi koniec dotknął podłoża
  (wymaga helperów budujących z pełną rotacją XYZ, nie tylko rot_z).
- Audyt automatyczny łapie koplanarność (pary ścianek <1 mm + realne nakładanie SAT),
  ale NIE łapie lewitacji. Warto dodać drugi audyt: "każdy prop ma kontakt" albo
  przynajmniej render-zbliżenie propów do ręcznej inspekcji przed pokazaniem.

## Kontekst odkrycia

Model stolarni (warsztat + lakiernia) budowany parametrycznym skryptem bmesh,
iteracja 2 po feedbacku Huntera. Wszystkie poprawki = zamiana szczelin na przenikania
+ przechyły; audyty z-fight nadal PASS.
