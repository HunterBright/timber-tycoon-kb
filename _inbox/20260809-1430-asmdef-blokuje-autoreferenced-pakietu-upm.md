---
type: lesson
project: Kerf - Sawmill Tycoon
suggested-category: engine/lessons
tags: [unity, asmdef, upm, pakiety, kompilacja, steamworks]
date: 2026-08-09
status: draft
---

# Własny asmdef unieważnia „autoReferenced" pakietu z Package Managera

## Objaw

Pakiet UPM zainstalowany poprawnie (widać go w logu builda, jego biblioteka się kompiluje),
a mimo to kod projektu nie widzi jego przestrzeni nazw:

```
error CS0246: The type or namespace name 'Steamworks' could not be found
```

Powtórzenie builda nie pomaga - to nie jest wyścig pierwszego importu, choć wygląda identycznie.

## Przyczyna

`"autoReferenced": true` w pakiecie działa **wyłącznie na zbiory wbudowane**
(Assembly-CSharp, czyli skrypty bez własnego pliku `.asmdef`). Gdy skrypty projektu mają
własny `.asmdef`, żadne automatyczne podpięcie nie zachodzi - taki zbiór widzi tylko to,
co ma jawnie wypisane w polu `references`.

Mylące jest to, że komunikat błędu brzmi identycznie jak przy braku pakietu, a log builda
pokazuje, że pakiet się pobrał i skompilował. Widać dwa prawdziwe zdania („pakiet jest",
„kompilator go nie widzi") i łatwo z nich wyciągnąć fałszywy wniosek o wyścigu importu.

## Naprawa

Dopisać nazwę zbioru pakietu do `references` w asmdef projektu:

```json
"references": [
    "UnityEngine.UI",
    "com.rlabrecque.steamworks.net"
]
```

Nazwa do wpisania to pole `"name"` z pliku `.asmdef` pakietu, nie nazwa katalogu.

## Jak rozpoznać w 30 sekund zamiast w 30 minut

Przy błędzie CS0246 na świeżo dodanym pakiecie pierwsze pytanie brzmi:
**czy moje skrypty mają własny asmdef?**

```
find Assets -name "*.asmdef"
```

Pusto - problem leży gdzie indziej. Coś jest - to prawie na pewno brakująca referencja.

## Pułapka towarzysząca

Gdy kompilacja pada, `-executeMethod` w ogóle się nie wykonuje, więc **raport builda
zostaje z poprzedniego uruchomienia**. Wygląda na „Succeeded" i kusi, żeby uznać build za
udany. Zawsze sprawdzać datę modyfikacji pliku raportu, nie samą jego treść.
