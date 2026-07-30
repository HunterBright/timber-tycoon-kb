---
title: Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-20'
project: Kerf - Sawmill Tycoon
tags:
- testy
- sonda
- red-proof
- dowod-porazki
- progi
- kalibracja
applies_to: []
source: ''
severity: medium
promoted: '2026-07-30'
---

# Dźwignia psująca jeden check może wyłączyć drugi - i jego tryb porażki zostaje nieudowodniony

## Kontekst

Projekt ma zasadę: **każdy nowy check sondy musi mieć udowodniony tryb PORAŻKI**.
Realizuje się to flagą "red-proof", która celowo psuje sprawdzaną rzecz, żeby zobaczyć,
że check umie zaświecić na czerwono.

Dodałem dwa powiązane checki pod jedną flagą:
- statyczny (`NPC/koła`): czy kod znalazł koła i czy mają kule kolizji
- dynamiczny (`NPC/koła-obrót`): czy koła realnie się kręcą podczas jazdy

Dwie dźwignie: (1) usuń kulę kolizji, (2) wyzeruj mnożnik obrotu.

## Co poszło nie tak

Kod ustawiał punkt odniesienia dla checku dynamicznego **dopiero po przejściu checku
statycznego**:

```
if (problemy.Count == 0) {
    Pass("NPC/koła", ...);
    baseline = ...;              // <- tylko tutaj
    if (redProof) wyzerujMnoznik();
}
```

Pod flagą dźwignia 1 psuła check statyczny, więc gałąź `if` nie wykonywała się.
Skutek: check dynamiczny **w ogóle się nie uruchamiał** - nie świecił ani na zielono,
ani na czerwono. Po prostu znikał z raportu.

Raport wyglądał na poprawny (jedna zamierzona porażka), a w rzeczywistości **tryb porażki
drugiego checku nie był udowodniony wcale**.

## Reguła

> Dźwignie red-proofu muszą działać **niezależnie od siebie**. Uzbrojenie checku
> (zapamiętanie stanu wyjściowego, rejestracja obserwatora) rób **przed** wszystkimi
> dźwigniami, nie w gałęzi sukcesu innego checku.

Test na siebie: pod flagą red-proof policz linie w raporcie. Jeśli któryś check
**zniknął** zamiast zaświecić na czerwono, dowód jest fikcyjny.

## Druga lekcja z tej samej sesji: próg musi przebijać zakłócenie

Check dynamiczny mierzył kąt obrotu koła. Pierwotnie planowany próg: 20 stopni.

Pod flagą red-proof (koła zamrożone) pomiar dał:
```
WheelLF=35 st.  WheelLR=0 st.  WheelRF=35 st.  WheelRR=0 st.
```

Przednie koła pokazały 35 stopni **z samego skrętu**, mimo że toczenie było wyłączone.
Przy progu 20 stopni przeszłyby jako "kręcą się". Check byłby fikcją dla połowy kół.

Maksymalny skręt w projekcie to 35 stopni, więc próg podniesiono do **45**.

> Gdy mierzysz wielkość, na którą wpływa więcej niż jeden mechanizm, próg musi być
> **wyższy niż maksymalny wkład mechanizmów, których NIE mierzysz**. Wartość tego
> zakłócenia weź z konfiguracji, nie z intuicji.

## Trzecia rzecz: modulo zjada pomiar jednorazowy

Kąt obrotu koła liczony jest modulo 360 stopni. Pojedynczy pomiar po przejechaniu
ustalonego dystansu może trafić w moment, w którym koło wróciło prawie do pozycji
wyjściowej po pełnych obrotach - i dać fałszywy alarm.

Rozwiązanie: **maksimum z wielu próbek** w trakcie całego przejazdu. Koło, które stoi,
ma to maksimum równe zeru niezależnie od liczby próbek.
