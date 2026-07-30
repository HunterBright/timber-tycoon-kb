---
type: lesson
project: Timber Tycoon
severity: medium
suggested-category: engine/lessons
tags: [unity, coplay, mcp, execute_script, tmpro, roslyn, editor-scripting]
date: 2026-07-10
status: draft
---

# Coplay execute_script nie kompiluje plików z `using TMPro;`

## Symptom
`execute_script` na pliku edytorowym zwraca mylący błąd:
`Could not find any resources appropriate for the specified culture ... CSharpResources.resources ... Coplay-v8.15.1`
(to zamaskowany komunikat diagnostyki Roslyna - realny błąd kompilacji jest nie do odczytania).
Ten sam plik kompiluje się w Unity bez błędów (`check_compile_errors` czysty).

## Przyczyna
Coplay `execute_script` kompiluje wskazany .cs **własnym Roslynem** z ograniczonym zestawem
referencji assembly. `Unity.TextMeshPro` nie jest wśród nich - każdy plik z `using TMPro;`
(albo innym typem spoza podstawowych assembly) wywala kompilację executora, mimo że projekt
jest zielony.

## Fix / obejście (zwalidowane)
Napisz mini-wrapper BEZ problematycznych usingów, który woła już skompilowaną metodę
z Assembly-CSharp-Editor (ta referencja jest dostępna):

```csharp
// scratchpad/RunShowroomSetup.cs - poza Assets/, Coplay przyjmuje absolutna sciezke
public static class RunShowroomSetup
{
    public static void Execute() { SetupFurnitureShowroom.Execute(); }
}
```

`execute_script(filePath: <sciezka wrappera>, methodName: "Execute")` - działa, bo TMPro
jest linkowane wewnątrz skompilowanego assembly projektu, a nie w kodzie kompilowanym przez executor.

## Reguła praktyczna
Skrypty edytorowe pod Coplay execute_script trzymaj wolne od `using TMPro;` / typów z paczek,
ALBO od razu odpalaj je przez wrapper. Plik wrappera może żyć w scratchpadzie (nie w Assets/) -
Unity go wtedy nie importuje.
