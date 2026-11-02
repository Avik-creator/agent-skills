---
name: prove
description: Proves a change works in the real running app before anyone says done - it starts the app, drives every flow the change touches the way a user or program would, compares the result with an answer key the builder did not write, and gets the verdict from a fresh agent, with evidence saved locally. On first use in a repo it sets up that repo's verifier (how to launch, check, drive and clean up the app, plus a feature map), and a git pre-push hook blocks a push that was not proved. Works with any coding agent. Use when the user says prove it, verify this works, check it in the real app, show me it works, set up verification for this repo, "the agent keeps saying done without testing", or before handing over or pushing any change with a runnable surface (a page, an API, a CLI, a job, an email). Not for writing test files - use testing; not for reviewing a diff - use `review`; not for a one-off browser task - use automation.
---

# Prove

"Done" means the change was seen working where users meet it, and checked against what right looks like. Code that compiles, tests that pass, and the builder's own reading of its diff are not proof.

## Quick start
1. Find the main checkout: `"$(git rev-parse --path-format=absolute --git-common-dir)/.."`. Everything lives in its `.prove/` folder.
2. No `.prove/verifier.md` yet: set it up first ([setup.md](references/setup.md)).
3. Run `scripts/keys check`. A changed answer key stops the run as BLOCKED until the user approves it again.
4. Work out what the change touches, drive it, and get a verdict from a fresh agent ([checker.md](references/checker.md)).
5. Write the receipt, clean up, and report each flow's verdict with its evidence.

## The folder (local only)
```text
.prove/
  verifier.md         this repo's launch, doctor, drive, evidence and cleanup steps
  features/<name>.md  one file per user-facing feature
  scripts/            start, health, stop and scripted checks
  keys/<task>/        answer keys: the proof plan, snapshots, expected outputs
  keys.lock           fingerprints of the keys the user approved
  runs/<YYYYMMDD-HHMMSS>-<task>/  receipt.md, screenshots, logs
```
Keep it out of git with the local exclude file: `echo '.prove/' >> "$(git rev-parse --git-common-dir)/info/exclude"` (once). Why: it changes no file in the repo, and nothing from a real project leaves the machine.

## Answer keys
- Every check compares against an answer key, ranked best first: old system, snapshot, independent tests, written spec, nothing. See [answer-keys.md](references/answer-keys.md).
- The user approves the keys, then run `scripts/keys lock`. Lock again only after the user says yes to a changed key. Why: the builder must never set its own bar.
- A key that disagrees with the app goes to the user. Never edit it to match the app.

## Run
1. **Scope.** From the diff, list what the change touches: new flows it adds, and existing flows it reaches directly or through shared code (schema, auth, navigation, a shared component). Check only those, never the whole app. A change of new files only also gets a short smoke test of the existing app.
2. **No runnable surface** (docs, comments, CI config, types only): write a receipt with `result NO-SURFACE` and say why.
3. **Launch and doctor** as `verifier.md` says. If the app won't start, the flow is BLOCKED with the exact error. Don't fix-loop a missing credential.
4. **Scripts first.** Run every scripted check: tests, API calls against saved responses, pixel diffs, database counts. Why: a script gives the same answer every time and shares no blind spot with the builder.
5. **Drive** each flow in scope like a user, with the methods in [methods.md](references/methods.md). Capture the action, the resulting state and every side effect.
6. **Verdict** from a fresh agent, as [checker.md](references/checker.md) says. Route on the checker's verdict, never on the driver's claim.
7. **Receipt and cleanup.** Write `runs/<run>/receipt.md`, stop only what this run started, then confirm the evidence is still there.

## Verdicts
- **PROVED:** every step showed its stated pass signal, checked by a fresh agent or a script.
- **BROKEN:** the app did something wrong. Report what, with evidence.
- **BLOCKED:** the check couldn't run (the app won't start, a credential is missing, a key changed). Never a weaker check in its place.
- **UNPROVEN:** no answer key, or only the builder checked it ("self-checked"). Never counts as PROVED.

## Rules
- Never edit the check, the harness or a key, and never restructure code just to make a check pass. If the key looks wrong, stop and ask.
- Never weaken a step so it passes. A failing precondition goes in the feature's Gotchas.
- A check you have never seen fail proves nothing. Every new check must catch a deliberately broken version once.
- Don't trust a dry-run or test mode by its name. Watch what it really touches.
- Real systems are read-only or off limits: no production data, no real payments or emails. Fake outside services and read the fake back.
- Evidence uses made-up test data only. Never capture a real user's name, email or records.
- A BROKEN verdict on work the user owns goes back to the builder: fix, prove again, at most 3 rounds, then stop and report. On someone else's change, report only.

## The push gate
After a yes, install the hook once per repo: `scripts/install-hook <path to this skill>/scripts`. It keeps any existing pre-push hook, the `ship` one included. A push is blocked unless the pushed commit has a receipt with `result PROVED` (not self-checked) or `NO-SURFACE`, and the keys match the lock. A new commit needs a new run. `PROVE_SKIP=1` skips one push; that is the user's call, never suggest it.

## Receipt format
```text
head <full commit sha>
date <ISO time>
checker subagent | cli:<name> | session | script | self
result PROVED | BROKEN | BLOCKED | UNPROVEN | NO-SURFACE
flow <name>: <verdict> - key <rung> - <evidence path> - <one line>
not covered: <flow> - <why> - <what would unlock it>
```
`result` is PROVED only when every flow is.

## Replies
Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

Lead with the overall result, then one line per flow: verdict, answer key and its rung, and the evidence path. Then what wasn't covered and why.

## References
- [references/setup.md](references/setup.md) - read the first time in a repo, or when adding a surface or a feature.
- [references/answer-keys.md](references/answer-keys.md) - read when choosing or saving an answer key.
- [references/methods.md](references/methods.md) - read when picking how to drive and check a surface.
- [references/checker.md](references/checker.md) - read before every verdict.
