# Proof

Read when proving a finding: the reviewer reads it, and you read it when re-running a proof. A finding is reported only with proof. Anything else goes to Couldn't prove or Judgement calls.

## First, try to rule it out

Before proving anything, read the callers, guards and tests that reach the changed code. Drop the finding if they rule it out. Why: most false findings come from reading a hunk without the code around it.

## What counts as proof

| Finding | Proof |
|---|---|
| A bug | A test you write in the copy that fails, showing the wrong result against the expected one. Or a command (a script, a `curl` to a local server) that shows it. |
| A missing test | A mutation the tests miss (see below). Name the test case that is missing. |
| A breach of the repo's rules | The rule quoted with its file and line, next to the code line that breaks it. |
| Spec: wrong behaviour | The intent bullet quoted, and a test or command that shows the behaviour differs. |
| Spec: a requirement missing | The intent bullet quoted, and a test or command that shows it isn't handled. A search that finds no code for it is supporting evidence, not proof on its own. |
| Security | A test or local request that shows the hole, such as user B reading user A's row. |
| Compatibility | A run with what the other side still sends or reads (the old payload, the old config, a client at the old version) that fails. For developer setup, the two lines quoted: the setup file that still uses the old name next to the code that reads the new one. |
| A feature gate leak | A test or local request, without the flag or the internal role, that reaches the gated code. |
| Performance | A measurement with its number and unit: a query count in a test, `EXPLAIN ANALYZE` on a local database, or a benchmark. |

Every proof command must run as written from the copy's path. Why: it gets re-run there, word for word.

## Mutations

To check whether the tests would catch a break:

1. Change one condition the diff adds: drop a negation, swap `&&` for `||`, change `<` to `<=`, or return early.
2. Run the tests that cover that code.
3. Restore the file: `git -C <copy> checkout -- <file>`.
4. Tests that still pass are a finding: Fix [tests], naming the missing test case, with the mutation and the run as proof.

Mutate one condition at a time, and only conditions the change adds. Why: one change at a time shows which condition is unguarded, and mutating old code reviews more than this change.

## Pre-existing or new

When a proof fails, run it again at the base:

```bash
START=$(git -C <copy> rev-parse HEAD)
git -C <copy> checkout --detach <base>
<the same proof command>
git -C <copy> checkout --detach "$START"
```

New, untracked test files survive the checkout. If the proof fails the same way at the base, mark the finding `(pre-existing)`. A test that fails at the base only because the code it calls doesn't exist yet doesn't count. Why: that is a build error, not the bug.

## Couldn't prove

Use it for real risks that can't be run here: a race that needs production load, a config only production has, a third-party service, or a test setup that won't start.

Write each one as: the risk, why it couldn't be proven here, and how to check it. Example: "Two workers can claim the same job. Couldn't prove because it needs two processes on a shared database, and the tests use SQLite. To check: run two workers against one shared Postgres and enqueue a single job."

A performance concern without a measurement goes here, not in Findings. Why: an unmeasured performance claim is a guess.

## Never

- Run against production, or call real third-party services. Use local or test setups.
- Edit the original repo, push, or open anything outward-facing from the copy.
- Run destructive commands (dropping databases, deleting files) outside the copy, or against shared data.
