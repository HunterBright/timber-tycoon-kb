---
title: 'Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"'
type: lesson
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Kerf - Sawmill Tycoon
tags:
- blender
- render
- color-management
- agx
- review
- low-poly
applies_to: []
source: ''
suggested-category: engine/lessons
---

# Rendery kontrolne do akceptacji kolorów: wyłącz AgX, użyj view transform „Standard"

## Problem
Rendery kontrolne modeli (6 nowych gatunków drzew) wychodziły wyblakłe — oliwkowa szarość
zamiast zieleni z zaakceptowanej karty kolorów. Wyglądało to na błąd materiałów, ale wartości
Base Color były poprawne.

## Przyczyna
Domyślny view transform Blendera (AgX, wcześniej Filmic) celowo desaturuje i spłaszcza kolory
(filmowy tone mapping). Do finalnego renderu artystycznego OK, ale do WERYFIKACJI wierności
kolorów względem specyfikacji — mylące. Dodatkowo mocne słońce + jasny world rozjaśniają
flat-color materiały low-poly.

## Rozwiązanie
W skryptach renderów kontrolnych: `scene.view_settings.view_transform = 'Standard'`
(+ look None, exposure 0, gamma 1) i stonowane oświetlenie (sun ~2.0, world ~0.5).
Wtedy piksele renderu odpowiadają wartościom Base Color i można porównywać z kartą kolorów 1:1.

## Reguła
Render „czy kolory się zgadzają" ≠ render „czy ładnie wygląda". Do pierwszego zawsze Standard.
