---
type: lesson
project: Timber Tycoon
suggested-category: engine/lessons
tags: [coplay, mcp, unity, execute_script, roslyn, editor-scripting, csharp]
severity: medium
time_lost: "~30 min"
date: 2026-06-09
status: draft
applies_to: [coplay-mcp, unity-editor-tooling]
---

# Coplay execute_script crashes opaquely on ANY C# compiler diagnostic (incl. a plain compile error)

## Problem
Calling Coplay MCP `execute_script` on a freshly-written editor `.cs` failed with an opaque
internal error that looked like a broken Coplay build, NOT a code problem:

```
Error executing script: Could not find any resources appropriate for the specified culture
or the neutral culture. Make sure "Microsoft.CodeAnalysis.CSharp...CSharpResources.resources"
was correctly embedded or linked into assembly "Coplay-v8.15.1" ...
  at Microsoft.CodeAnalysis.CSharp...ErrorFacts.GetMessage (...)
  at Coplay.MCP.ScriptExecutor.CompileCode (System.String code)
```

The script never ran (`Core()` never started). No `[MyTag]` console output appeared. Earlier
scripts the same day ran fine, so it looked like the bridge had spontaneously broken
(a Play-Mode cycle had happened in between, which made "domain reload broke Coplay's Roslyn"
a tempting but WRONG hypothesis).

## Root cause
The submitted code contained one ordinary compile error (`System.Math.Ceil` — which does not
exist; it's `Math.Ceiling`). Coplay compiles via Roslyn and, on encountering ANY diagnostic
(error OR warning), calls `ErrorFacts.GetMessage` to format the message text. That call needs
the satellite resource assembly `CSharpResources.resources`, which is missing/unloadable in the
Coplay build — so formatting the diagnostic throws, masking the real diagnostic entirely.

Net effect: **Coplay only runs code that compiles with ZERO diagnostics.** The moment your
code has even one warning or error, you get the misleading "could not find resources" crash
instead of the actual compiler message — so you can't see what's wrong.

## Solution
Don't debug blind against Coplay's compiler. Get the real diagnostic from Unity, then trigger
through a trivial wrapper:

1. Put the real logic in **`Assets/Editor/Foo.cs`** so Unity's own compiler builds it. Then
   `check_compile_errors` returns the EXACT error (`Assets\Editor\Foo.cs(319,29): error CS0117:
   'Math' does not contain a definition for 'Ceil'`). Fix until green.
2. Trigger it through a **trivial, warning-free reflection wrapper** placed outside `Assets/`,
   invoked via `execute_script`:

   ```csharp
   public static class RunFoo {
       public static void Go() {
           System.Type t = null;
           foreach (var a in System.AppDomain.CurrentDomain.GetAssemblies()) {
               t = a.GetType("Foo"); if (t != null) break;
           }
           var mi = t.GetMethod("Go", System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Static);
           mi.Invoke(null, null);
       }
   }
   ```

   Coplay compiles ONLY this wrapper (clean → no `GetMessage` → no crash); it calls the
   Unity-compiled class by reflection. Reflection-by-name also avoids any compile-time
   dependency, so the wrapper compiles even if Coplay's Roslyn doesn't reference
   `Assembly-CSharp-Editor`.
3. Bonus: add `[MenuItem(...)]` on the entry methods so a human can run it if MCP is unavailable.

Confirm the bridge is healthy with a one-line clean "ping" script (`Debug.Log("ping")`); if the
ping runs but your script doesn't, the difference is a diagnostic in your script.

## What didn't work
- Re-running the same script via `execute_script` (same opaque crash every time).
- Treating it as "Coplay bridge broke after Play Mode exit" — it was never the bridge.
- Trying to read the real error from `get_unity_logs` — nothing is logged; the crash is in the
  compile step before any user code runs.

## Transferability
Applies to ANY Unity project driven by Coplay MCP `execute_script`, regardless of genre. The
"compile in Assets/Editor for the real error + trigger via a clean reflection wrapper" pattern
is the reliable way to run non-trivial editor C# through Coplay without flying blind. The
general principle — when a tool's compiler crashes formatting its own diagnostics, compile the
code somewhere that surfaces the real message — generalizes beyond Coplay.

## Related
- Project memory: how Coplay execute_script is run in Timber Tycoon (file-on-disk pattern).
