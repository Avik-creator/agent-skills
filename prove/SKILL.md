---
name: prove
description: Proves a change works in the real running app before anyone says done - it starts the app, drives every flow the change touches the way a user or program would, compares the result with an answer key the builder did not write, and gets the verdict from a fresh agent, with evidence saved locally. On first use in a repo it sets up that repo's verifier (how to launch, check, drive and clean up the app, plus a feature map), and a global git pre-push hook, turned on automatically, blocks any push that was not proved. Works with any coding agent. Use when the user says prove it, verify this works, check it in the real app, show me it works, set up verification for this repo, "the agent keeps saying done without testing", when a push is blocked by "prove:", or before handing over or pushing any change. Not for writing test files - use testing; not for reviewing a diff - use `review`; not for a one-off browser task - use automation.
---

# Prove

"Done" means the change was seen working where users meet it, and checked against what right looks like. Code that compiles, passing tests, and the builder's reading of its own diff are not proof.

## Quick start
1. Turn on the global hooks: run `<skill dir>/../_hooks/install`, then `install check`, without asking. Tell the user in one line if it turned them on, or why they can't run in this repo (`ship`'s hook.md covers hook managers). Why: the user wants no push to skip a check.
2. `D=$(scripts/init)` prints the `.prove/` folder in the main checkout, creates it, and keeps it out of git.
3. No `$D/verifier.md`: set it up first ([setup.md](references/setup.md)).
4. Find the task's keys in `$D/keys/<task>/`. None: build them with the user ([answer-keys.md](references/answer-keys.md)), then `scripts/keys lock <task>`. Run `scripts/keys check <task>`; a changed key is BLOCKED until the user approves it again.
5. Scope, drive and get a verdict from a fresh agent ([checker.md](references/checker.md)). Write the receipt, clean up, and report.

## The folder (local only)
```text
.prove/
  verifier.md         this repo's launch, doctor, drive, evidence and cleanup steps
  features/<name>.md  one file per user-facing feature
  scripts/            start, health, stop
  keys/<task>/        answer keys: plan.md, snapshots, expected outputs, locked checks
  locks/<task>.lock   fingerprints of the keys the user approved
  runs/<YYYYMMDD-HHMMSS>-<task>/  receipt.md, screenshots, logs
```
Why local: nothing from a real project leaves the machine, and teammates get no new folder.

## Run
1. **Scope.** From the diff, list what the change touches: new flows, and existing flows it reaches directly or through shared code (schema, auth, navigation, a shared component). Check only those. Why: checking the whole app is slow and hides what the change did. A change of new files only adds a short smoke test, keyed to the base commit.
2. **No runnable surface** (docs, comments, CI config, types only): receipt with `result NO-SURFACE` and the reason, after a fresh agent (not `script` or `self`) reads the diff and agrees nothing runnable changed. Why: otherwise the builder could skip proof by saying there is nothing to prove.
3. **Launch and doctor** as `verifier.md` says. If the app won't start, the flow is BLOCKED with the exact error. Why: no code change fixes a missing credential, so there is nothing to loop on.
4. **Scripts first:** tests, API calls against saved responses, pixel diffs, stored-state counts. Why: a script gives the same answer every time and shares no blind spot with the builder.
5. **Drive** each flow like a user, with [methods.md](references/methods.md). Capture the action, the resulting state and every side effect.
6. **Verdict** from a fresh agent, as [checker.md](references/checker.md) says. Route on the checker's verdict, never on the driver's claim.
7. **Receipt and cleanup:** write `runs/<run>/receipt.md` as checker.md shows, stop only what this run started, then confirm the evidence is still there.

## Verdicts
- **PROVED:** every flow showed its pass signal, judged by a fresh agent or a locked script.
- **BROKEN:** the app did something wrong. Report what, with evidence.
- **BLOCKED:** the check couldn't run. Never put a weaker check in its place. Why: a weaker check that passes reads as proof.
- **UNPROVEN:** no answer key, only the builder judged it, or a flow wasn't covered and the user didn't agree to that.

## Rules
- Only the user changes a key. A key that disagrees with the app goes to the user. Why: the builder must never set its own bar.
- `verifier.md` changes only through the drift rule in setup.md, and each change is reported. The builder never loosens a check, edits the harness, or restructures code to pass. Why: each one turns a failure into a pass without fixing anything.
- A check you have never seen fail proves nothing. Every new check catches a deliberately broken version once, as setup.md shows.
- Don't trust a dry-run or test mode by its name; watch what it touches. Why: some still send email or call the network.
- Real systems are off limits: no production data, real payments or real emails. Fake outside services and read the fake back.
- Evidence uses made-up test data only. Why: a screenshot can end up in a PR.
- BROKEN on work the user owns goes back to the builder: fix, prove again, at most 3 rounds, then stop and report. Why: a fourth try rarely works where three didn't. On someone else's change, report only.

## The push gate
The global `pre-push` blocks a push unless the pushed commit has a receipt with `result PROVED` (with a named checker and a locked task whose keys still match, and no unagreed gaps) or `result NO-SURFACE` (with a fresh-agent checker). Every new commit needs a new run. `PROVE_SKIP=1` skips one push; that is the user's call, never suggest it.

## Example
User: "Prove the new invite flow." First run in the repo.
1. Hooks on. No `verifier.md`: set it up (web plus API), and proved it catches a planted bug.
2. Keys: the spec from the PR, and screenshots of the members page from the base commit. The user approved; locked as `invite-flow`.
3. Scope: the invite flow (new), and the members page (shared component). The API check passed; a subagent judged both screens.
4. Reply: "PROVED. Invite: spec (4), runs/20261103-142000-invite-flow/flow-1.txt. Members page: snapshot (2), 0% difference at 3 widths. Not covered: the invite email, because there is no local email catcher (agreed)."

## Replies
Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

Lead with the overall result, then one line per flow: verdict, answer key and its rung, and the evidence path. Then what wasn't covered and why.

## References
- [references/setup.md](references/setup.md) - read the first time in a repo, when adding a surface or feature, or when a step drifts.
- [references/answer-keys.md](references/answer-keys.md) - read when choosing, saving or locking an answer key.
- [references/methods.md](references/methods.md) - read when picking how to drive and check a surface.
- [references/checker.md](references/checker.md) - read before every verdict, and for the receipt format.
