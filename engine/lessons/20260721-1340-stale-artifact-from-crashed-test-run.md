---
title: Crashed test run leaves the PREVIOUS report on disk and reads as "my code never got built"
type: lesson
status: active
confidence: medium
verified: ''
date: '2026-07-21'
project: Kerf - Sawmill Tycoon
tags:
- testing
- build-gate
- ci
- artifacts
- false-negative
- diagnostics
applies_to:
- any test harness that writes a report file at the end of the run
source: ''
severity: high
time_lost: ~15 min
promoted: '2026-07-30'
---

# Crashed test run leaves the PREVIOUS report on disk and reads as "my code never got built"

## Problem
Added three new checks to the project's in-build smoke probe, rebuilt, ran the probe, read the
report file. The report showed the OLD check count and none of the new sections - as if the new
code had never reached the build.

Spent 15 minutes chasing the wrong cause: hunting for a duplicate class in another assembly,
checking asmdef boundaries, even decoding the shipped DLLs to look for my own string literals.
The literals were there. The source was correct. The build was fresh.

The report file was three hours old. The probe process had died mid-run and the report - written
only at the very END of a successful run - was still the one from the previous session.

## Root cause
The harness writes its report as its last action:

    WriteReport();
    if (!Application.isEditor) Application.Quit(failed == 0 ? 0 : 1);

Any crash, hang or early exit before that line leaves the previous run's artifact untouched. The
artifact therefore has two indistinguishable meanings:
  - "this run finished and here is its result"
  - "this run died and you are looking at an older run"

Worse, the stale artifact is *plausible*: it is a well-formed report of the same program, so
nothing about its content signals staleness. It actively misleads, because the older run
predates the change under test and so "proves" the change is absent.

## Solution
Delete the artifact before every run, and treat "no artifact" as a first-class failure:

    Remove-Item .\Builds\Win64\MeshAudit_Result.txt -Force -ErrorAction SilentlyContinue
    & .\Builds\Win64\Game.exe -meshaudit -logFile player.log
    if (-not (Test-Path .\Builds\Win64\MeshAudit_Result.txt)) { "RUN DIED - no report" }

Two cheap secondary signals that would have caught it immediately:
  - **Wall-clock duration.** A full run takes ~2.5 min here; the bad run returned in ~30 s. A
    suspiciously fast "pass" is a crash until proven otherwise.
  - **Artifact mtime vs. binary mtime.** If the report is older than the built binary, it cannot
    describe that binary.

Better still, make the harness stamp the run: write the report (or a run header) at START with
"INCOMPLETE", and overwrite at the end. Then a dead run leaves evidence of itself instead of
someone else's success.

## Second mechanism, same trap: the command returns before the work finishes

Hit this twice in one session, the second time from a different direction. `Unity.exe -batchmode
-quit -executeMethod ...` **returns to the shell before the build has finished writing**. So:

    & Unity.exe -batchmode -quit -executeMethod Build...     # returns in ~40 s
    Get-Content build_report.txt                             # report from THREE HOURS AGO
    # -> "the build produced nothing / my code is not in it"

Identical symptom, identical wrong conclusion, different cause: there the artifact was stale
because the run died; here because the run had not started writing yet. Both are "the file on
disk does not describe the thing you just did".

The fix generalises: **never treat process exit as completion for a tool that reports through a
file.** Wait for the tool's own completion marker instead:

    do { Start-Sleep 5
         $ok  = Select-String build.log -Pattern '\[BUILD\] OK' -Quiet
         $err = Select-String build.log -Pattern 'error CS|\[BUILD\] FAIL' -Quiet
    } while (-not $ok -and -not $err -and (Get-Date) -lt $deadline)

Note the loop waits for success **or** failure **or** a deadline. Polling only for the success
marker reintroduces the original bug in loop form: a dead build would spin until timeout and then
be indistinguishable from a slow one.

## Third variant worth naming: a concurrent session poisons the run

Same session again: the in-build probe went red (5 failures, all asserting "fresh game state")
purely because a human was playing another copy of the same game at that moment - the two
processes share per-user state (PlayerPrefs / save directory). Re-running with nothing else open
was clean. So before diagnosing a red run, check not only *is this artifact fresh* but *was this
run alone*. Cheap guard: have the harness refuse to start, or at least log loudly, if another
instance of the product is running.

## What didn't work
- **Trusting the exit code.** This project already knows exit codes lie here (the exe has crashed
  on shutdown with a complete report). The reflex became "read the file, not the code" - but the
  file has its own failure mode, and nobody had written that half down.
- **Reasoning from the build outputs.** Verifying the source, the asmdef layout, and even
  grepping the compiled assembly for the new string literals all came back green and only
  deepened the confusion, because the question being asked ("is my code in the build?") was the
  wrong one. The right question was "is this report from this run?".
- **Looking for a duplicate class in another assembly.** Pure red herring, costed the most time.

## Transferability
Nothing here is engine- or language-specific. Any harness whose result lives in a file written on
the success path has this hole: pytest with `--junitxml`, a coverage report, a benchmark CSV, a
generated build manifest, a Playwright HTML report. The pattern is:

  **An artifact that is only overwritten on success cannot distinguish "failed" from "not run".**

The fix is the same everywhere: delete before, require after, and prefer harnesses that write an
"in progress" marker up front.

Corollary for AI-assisted work specifically: an agent reading a stale artifact will confidently
build a whole wrong theory on top of it, because the artifact looks like fresh evidence. Cheap
freshness assertions (mtime, duration, run id) are worth more than they cost.

## Related
- [[gate-must-have-provable-failure-mode]]
- Project rule already in CLAUDE.md: read the report FILE, not `$LASTEXITCODE` - this lesson adds
  the missing second half: and check that the file is FRESH.

<!-- POWIAZANE:auto -->
## Powiazane

*Dobrane automatycznie po wspolnych tagach. Kolejnosc wedlug sily zwiazku.*

- [[20260722-1652-relative-only-test-blind-to-common-mode-error|Test porownujacy instancje MIEDZY SOBA jest slepy na blad wspolny (common-mode)]] - wspolne: build-gate, testing
<!-- /POWIAZANE:auto -->
