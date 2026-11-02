---
name: review
description: Reviews a code change through a fresh subagent that proves each finding - a failing test, a reproduction or a mutation the tests miss for a bug, the quoted rule for a rule breach - and reports proven findings apart from unproven risks and opinions. Checks the change against what it was meant to do (decisions agreed with the user, a design write-up, the PR or issue) and against the repo's own rules. Runs on its own before handing over any change that affects behaviour, and on request for a branch, commit range, pull request or pasted diff. Use when finishing a code, SQL, migration or config change, or when the user asks to review, check or audit a diff, branch or PR, says "review since X", "look over this PR", "what could this break" or "is this ready to merge", or asks for a strict, deep, harsh or thermo-nuclear code quality review. Not for checking a change in a real browser - use automation; not for writing tests - use testing; not for a repo-wide cleanup - use refactor.
---

# Review

A review is worth what it proves. A fresh reviewer reads what the code does, not what you meant it to do, and every finding comes with its proof: something that ran, or the exact rule it breaks.

## Quick start

1. Pin the base and get the diff. Stop on a bad ref or an empty diff.
2. Collect what the change should do, and the repo's rules.
3. Brief a fresh subagent with [reviewer-brief.md](references/reviewer-brief.md).
4. Re-run every proof it cites.
5. Report in chat. Then fix (your own work) or offer to post (someone else's).

## When it runs

- **Before you say "done"** on any change that affects behaviour: code, SQL, migrations, config that changes what runs. Why: this is the self-review that comes before the user's own review.
- **Skip** changes that don't (docs, comments, typo fixes), and say so: "Review skipped: docs only." Someone else's branch or PR: only when the user asks.
- **Strict**: when the user asks for a strict, deep or thermo-nuclear quality review, also run [strict.md](references/strict.md). Its Structure section goes after Findings.

## Workflow

### 1. Pin the base
- Use the base the user names. For your own work, use the merge-base with the default branch, and include uncommitted changes: `git diff $(git merge-base HEAD origin/main)`, with the repo's default branch in place of `main`. New files that haven't been added are missing from that diff; list them with `git status --short` and include them. A pasted diff has no base: review it as text, and prove only what the code in this repo lets you run.
- Check both ends resolve (`git rev-parse`) and there is something to review: a non-empty diff or new files. Why: a bad ref should fail here, not inside a subagent.
- `git diff --shortstat` plus the new files' lines gives the size, which decides step 3.
- Never check out a PR over the user's working tree. Use a separate `git worktree` for its head. Why: checking out would swap their files and may lose their uncommitted work.

### 2. Collect intent and rules
- **What it should do**: use every source that exists, and where they conflict the earlier one wins: decisions agreed with the user in chat, a design write-up, the PR description and linked issue. Someone else's PR with none of these: ask the user what it is meant to do.
- Write the intent as a short bullet list. Why: the reviewer can't see your chat; the brief is all it gets.
- **How it should be written**: the repo's rule files (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, style guides), then the smells in [checks.md](references/checks.md) as judgement calls. The repo's rules win where they disagree.

### 3. Brief a fresh reviewer
- One fresh subagent, never your session history. Why: you are anchored on what you meant to write.
- Over about 400 changed lines, run two in parallel: one for intent, one for rules. Over about 1,000, also suggest splitting the change.
- The reviewer runs the tests and the linter itself, and proves findings as [proof.md](references/proof.md) says. Anything it writes (a new test, a mutation) goes in a copy you make for it, never the code under review.
- Run against local or test setups only, never production. Before running code from a PR by someone outside the team or from a fork, ask the user. Why: its tests and scripts run on this machine with your access.

### 4. Check the proof
- Re-run each command the reviewer cites. A proof that doesn't reproduce moves the finding to Couldn't prove. Why: a wrong reviewer sounds as sure as a right one.
- If you think a finding is wrong, show your proof and list it as Disputed with the reason. Never drop it quietly.
- On a PR, only now read its existing comments, as [breakage.md](references/breakage.md) says.
- Then remove the copies you made (`git worktree remove --force <path>`, only those paths). Why: each one is a full checkout left on disk.

### 5. Report
Use the format below, in chat. Zero findings is a valid result; don't invent one. Leave out an empty section.

### 6. After the report
- **Your own work**: fix every Blocker and Fix, then re-run each proof, which should now pass. Keep a proof test as the regression test in your change. Why: it is a test already seen failing before the fix. Large fixes get a fresh review of the new code. Ask before a fix that changes a schema, an API, security behaviour or what users see.
- **Judgement calls and Couldn't prove**: list them; act only if the user says so.
- **Dead code** the change leaves behind: list it, delete after a yes.
- **Simplification**: if the reviewer flagged signals from [simplify.md](references/simplify.md), offer a pass on your changed lines; run it only after a yes.
- **Someone else's PR**: report only, never edit their code. Post only after a yes, as [feedback.md](references/feedback.md) says, and never mark it approved or "changes requested" unless asked. Why: a post notifies people and stays on the record, and that status can block or unblock a merge.

## Output format

```
Review of <base>..<head>: <N> files, +<A>/-<D>. Reviewer: 1 fresh subagent.
Ran: <test command> (<result>), <lint command> (<result>), or what couldn't be run and why

Findings (<n>)
1. Blocker [<check>] <file>:<line> - <the defect, one line>
   Proof: <command> -> <what it showed>, or <rule file>:<line> quoted
2. Fix [<check>] ...

Couldn't prove
- <file>:<line> - <the risk>. Not proven because <reason>. To check: <how>.

Judgement calls
- <file>:<line> - possible <smell>: <one line>.

Disputed
- <finding> - <why you disagree, with your proof>.
```

- **Blocker**: broken behaviour, a security hole, or data loss. **Fix**: any other proven finding. Blockers first.
- `<check>` is one of correctness, compatibility, security, performance, tests, architecture, readability, standards, spec.
- Mark a defect that already existed at the base as `(pre-existing)`. Why: it is real, but not this change's fault.

## Example

User: "Review PR 214." Its description says "retry failed webhooks up to 3 times".
1. `gh pr diff 214`: 3 files, +120/-8. One reviewer.
2. The reviewer reports a retry with no idempotency key. It proves it with a test that delivers the same webhook twice: two rows are written.
3. You re-run that test: two rows. Report: "Blocker [correctness] webhook.go:58 - a retry writes a duplicate row. Proof: go test -run TestRetryTwice -> 2 rows, want 1." You don't post it until the user says so.

## References

- [references/reviewer-brief.md](references/reviewer-brief.md) - read before briefing the reviewer.
- [references/checks.md](references/checks.md) and [references/breakage.md](references/breakage.md) - the checks, the smells, and what the change breaks outside the diff; the reviewer reads both.
- [references/proof.md](references/proof.md) - how to prove a finding, and mutating safely on a copy.
- [references/feedback.md](references/feedback.md) - read before posting on someone else's PR.
- [references/dependencies.md](references/dependencies.md) - read when the change adds or upgrades a dependency.
- [references/simplify.md](references/simplify.md) - the reviewer reads its signals; read it all before a simplification pass.
- [references/strict.md](references/strict.md) - read on a strict or thermo-nuclear request; the reviewer reads it too.
