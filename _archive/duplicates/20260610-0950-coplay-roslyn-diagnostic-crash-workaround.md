---
title: Coplay execute_script crashes on ANY compile diagnostic — compile via Unity + reflection trigger instead
type: lesson
status: superseded
confidence: low
verified: ''
date: '2026-06-10'
project: Kerf - Sawmill Tycoon
tags:
- coplay
- mcp
- execute_script
- roslyn
- unity-editor
- workaround
- claude-code
applies_to: []
source: ''
supersedes: 20260609-1045-coplay-execute-script-roslyn-diagnostic-crash
---

# Coplay execute_script crashes on ANY compile diagnostic — compile via Unity + reflection trigger instead

## Symptom
Coplay MCP `execute_script` on a ~400-line editor script fails with:

```
Could not find any resources appropriate for the specified culture...
"Microsoft.CodeAnalysis.CSharp.<guid>CSharpResources.resources" ... assembly "Coplay-v8.15.1"
at Microsoft.CodeAnalysis.CSharp.ErrorFacts.GetMessage
at Coplay.MCP.ScriptExecutor.CompileCode
```

The script never runs. The REAL compile diagnostic is unrecoverable — Coplay's embedded Roslyn is missing its localized resources satellite, so the moment it tries to *format any diagnostic message* (error **or possibly warning**), it throws this masking exception instead. Small "warning-free" scripts run fine (observed across many sessions); complex scripts that trip any diagnostic die opaquely.

## Workaround (verified 2026-06-10, Coplay v8.15.1, Unity 6000.x)
1. Put the real script in `Assets/Editor/<Name>.cs` — Unity compiles it with full, readable diagnostics (`check_compile_errors` shows real file/line/message; fix until clean).
2. Run it via a tiny, deliberately boring trigger file (outside Assets, e.g. `_TempEditor/`) passed to `execute_script`:

```csharp
public static class MyTool_Trigger
{
    public static string Execute()
    {
        var type = System.Type.GetType("MyTool, Assembly-CSharp-Editor");
        if (type == null) return "[Trigger] FAIL: type not compiled yet";
        var m = type.GetMethod("Execute");
        return m.Invoke(null, null) as string ?? "(null)";
    }
}
```

Reflection (rather than a direct call) keeps the trigger compilable even while the Assets script is still compiling, and immune to assembly-reference quirks.

## Rules of thumb
- Keep dynamic `execute_script` payloads SHORT and warning-free (no unused locals, no obsolete APIs).
- Anything non-trivial → Unity-compiled `Assets/Editor` + reflection trigger. The editor script can be left untracked or deleted after use.
- A self-guarding script (pre-checks, abort paths, report file written on EVERY exit) pairs well with this: the report file's absence then proves zero execution after a bridge failure.
