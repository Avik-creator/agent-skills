# Sources

## The author's skill
`the-cause`, pasted in chat on 2026-11-02, is the core: the Scope, Before writing, The report and Replies sections are kept word for word except for these changes:
- `no-slop` is now `coding`. `cases-to-run` becomes `test-cases` for manual cases, while the reproduction test goes to `testing` through `coding`.
- Scope adds what may run without an edit (a scratch reproduction outside the tree, `git bisect` on a clean tree) and what needs a yes (a temporary log line). The author agreed to this on 2026-11-02.
- The report adds three items: the handoff, the signal and test level that would have caught it, and scratch files kept.
- Replies adds the stopping note.

Original text:

```
---
name: the-cause
description: >-
  Use when asked to diagnose, debug, or find why something fails. Not for
  writing the fix or the test.
---

# The cause

## Scope

Read only the path the bug touches, its callers, and the logs, traces, or connected sources needed to prove the cause. Do not edit the code. The fix goes to `no-slop`. The regression case goes to `cases-to-run`.

Name the source. A local workaround is not a cause, and a cause without evidence stays a hypothesis.

## Before writing

Ask every blocking question in one turn. Never invent a fact. Put an unknown under Open questions and name what it blocks. Replies are 1–2 lines. Write a paragraph only when the user asks.

Ask what would change the reproduction or the expected behavior.

## The report

- The shortest reproduction: the steps, the data, and the environment when it changes the result.
- Expected behavior and actual behavior.
- Where it happens: the file, the function, and the condition.
- The root cause, backed by the code or a log line. A cause without evidence stays marked as a hypothesis.
- What was checked and ruled out.

Name the symptom, then the source. A local workaround is not a cause.

## Replies

Say the cause, or the hypothesis still open.
```

## Upstream

| Source | Licence | Taken |
|---|---|---|
| qa-skills-main `skills/bug-reproduction` (SKILL.md and intake, bisect, determinism, ticket-writeback) | MIT | Repro spec rows, shrinking one cut at a time, pinning time, randomness and network, the four can't-reproduce cases, one targeted check for bisect, exit 125 for unbuildable and flaky commits |
| cc-skills-golang `golang-troubleshooting` (SKILL.md, `methodology.md`) | MIT | Compare with working code, trace back to where the value was made, undo a failed probe, three failures means a wrong model, read the callers, causes outside the code, a library blamed only with a minimal repro |
| wshobson/agents `agent-teams` (`parallel-debugging`, `hypothesis-testing.md`, `team-debugger.md`) | MIT | The six categories, evidence grades, confirmed/disproved/inconclusive, a compound cause, recording evidence against |
| wshobson/agents `developer-essentials/debugging-strategies` | MIT | The working-versus-broken comparison (also in golang-troubleshooting) |
| addyosmani/agent-skills `debugging-and-error-recovery`, wshobson/agents `debugging-toolkit`, `error-debugging`, `incident-response` (2026-11-02 sweep, items 1-15 and 17) | MIT | Error text is data, keep the evidence, name the layer, kinds of failure, stack reading, timeline and upstream, observed/inferred/unknown, read-only isolation, cumulative `curl -w`, mitigation as a proposal, temporary logs, stripping ids before comparing, the stopping note |
| qa-skills-main `quality-postmortem` | MIT | The test level that should have caught it |
| agent-skills `evals/cases/debugging-and-error-recovery.json` and fixtures | MIT | Kept as eval material for the with/without test |

## Dropped
- The bisect wrapper in bug-reproduction `references/bisect.md:65`: `if ! npm ci && npm run build` never runs the build after a good install, so an unbuildable commit isn't skipped. Replaced with a tested script.
- Confidence bands (">80%", "50-80%") and the triage skill's "85% accuracy": these are invented.
- Defense in depth at four layers (golang-troubleshooting methodology step 10). It clashes with the simplicity rule, and the fix is `coding`'s job.
- The Go-only material (race detector, pprof, goroutine dumps, Delve) belongs in the go skill later.
- Everything the 2026-11-02 sweep listed under conflicts and skip: production edits, invented thresholds, vendor tools as the method, fiction in examples.
- Ticket write-back: the report stays in chat unless the user asks.

## Verified
- `git help bisect` on git 2.54.0: 1-127 except 125 is bad, 125 is a skip, any other code stops the bisect. "There are only 'skip'ped commits left to test" was found in the binary.
- `man curl` 8.7.1: each `time_*` value is measured from the start, and `time_starttransfer` includes server time.
- The step script was tested in a scratch repo with eight commits:
  - Exit codes: good gave 0, bad gave 1, and unbuildable, flaky and can't-judge each gave 125.
  - Bisect named the commit that introduced the bug.

## Refresh
1. Re-read the upstream files above for changes.
2. Re-run the step-script test, and re-check `git help bisect` exit codes on the installed git.

## Added 2026-11-02 (engineering-method split)
From the author's `engineering-method` §1 and §4:
- A problem report ends with the report, and the fix waits for a yes.
- "Consistent with" is not "caused by".
