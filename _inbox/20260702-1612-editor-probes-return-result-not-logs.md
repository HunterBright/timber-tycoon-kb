---
title: 'Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log'
type: pattern
status: draft
confidence: low
verified: ''
date: '2026-07-02'
project: Timber_Tycoon
tags:
- unity
- mcp
- coplay
- editor-scripting
- diagnostics
- play-mode
applies_to: []
source: ''
suggested-category: workflow/patterns
---

# Sondy edytorowe przez MCP: zwracaj raport jako Result (string), nie przez Debug.Log

## Kontekst
Autonomiczna praca w Unity przez MCP (Coplay `execute_script`): skrypty diagnostyczne („sondy") odpalane w Edit/Play Mode do sprawdzania stanu sceny, magazynu, kolejek NPC itd.

## Problem
Bufor logów konsoli czytany przez MCP (`get_unity_logs`) potrafi **zamarznąć po wyjściu z Play Mode** — kolejne `Debug.Log` z sond edytorowych nie pojawiają się w odczycie, mimo że skrypt wykonał się poprawnie (Success:true). Diagnoza w ciemno.

## Wzorzec
Metoda sondy zwraca `string` — MCP oddaje go bezpośrednio jako `Result` wywołania, z pominięciem konsoli:
```csharp
public class MyProbe
{
    public static string Execute()
    {
        var sb = new StringBuilder("[PROBE]\n");
        sb.AppendLine($"stan: {...}");
        return sb.ToString(); // trafia do Result — zawsze czytelne
    }
}
```
Dodatkowe zalety: raport atomowy (nie ginie między innymi logami), łatwe asercje „oczekiwane vs faktyczne" w jednej odpowiedzi, działa identycznie w Edit i Play Mode. `Debug.Log` zostawiać tylko jako ślad wtórny.

## Kiedy stosować
Każda sonda/diagnostyka odpalana przez MCP, szczególnie po przejściach Play↔Edit. Długie przebiegi w Play Mode (smoke testy) nadal mogą logować do konsoli — tam bufor działa; zamarza dopiero po wyjściu z Play.
