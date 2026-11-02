---
name: ship
description: Takes a finished change to an open pull or merge request, then stops so the user merges and releases. Runs self-review and a done gate, scans the diff for a lowered bar (new suppressions, skipped or deleted tests, removed assertions, stubs), commits in logical pieces, then interviews the user about the feature and checks each answer against the code before anything is pushed; a global git pre-push hook, turned on automatically, blocks any push without the interview and a prove receipt. Use when the user says ship it, ready to ship, wrap this up, we're done, push this, open a PR or MR, create a merge request, or asks for a rollback plan, a deploy or release checklist, a version bump, a tag or a changelog, or to clean up after a merge. Not for reviewing someone else's PR - use `review`; not for adding telemetry or alerts - use observability.
---

# Ship

Nothing leaves this machine until the change is checked, the bar is intact, and the user can explain what is going out. The agent prepares and pushes the branch; the user merges, deploys and tags.

## Quick start

1. Check the state, and turn on the global hooks.
2. Self-review with `review`, then the done gate and `scripts/bar-guard`.
3. Show the commit plan; commit after a yes.
4. Interview the user ([interview.md](references/interview.md)) and write the record.
5. Show the push and the PR or MR; run them after a yes. Stop.

## Workflow

### 1. Check the state
- On the default branch: propose a branch name and create it after a yes. Why: nothing is ever pushed to the default branch.
- Find the host from `git remote -v`: GitHub uses `gh`, GitLab uses `glab`. Say "PR" or "MR" to match.
- Run `<skill dir>/../_hooks/install`, then `install check`, without asking, as [hook.md](references/hook.md) says. Tell the user in one line if it turned the hooks on, or why they can't run in this repo. Why: the user wants no push to skip a check.

### 2. Self-review and the done gate
- Run `review` on the change if it has not run since the last edit. Fix its Blockers and Fixes as it says.
- Walk the done gate in [done.md](references/done.md). Report each item as done, not applicable (with the reason), or missing.
- Run `scripts/bar-guard`. Each hit is shown to the user with a reason or fixed; none is dropped quietly. Why: a red check turned green by silencing it is the cheapest way to lower the bar.

### 3. Commit
- Group the changes into commits, one logical change each, with refactors apart from behaviour. Show the plan: files and message per commit.
- After a yes, commit. Stage only that commit's files by name, never `git add -A` or `git add .`. Why: a stray file rides along unseen. A hedged answer ("I guess", "probably") is not a yes. Before each commit, run `git diff --cached -i -G'password|secret|api_key|token' --name-only`; any output is read before going on.
- Messages: a short subject, and a body saying why. No AI attribution trailer. Never `--no-verify`. Why: it skips every hook, the linters too.

### 4. Interview
Run [interview.md](references/interview.md) on the committed change. When it ends, write the record so the hook knows:
```sh
rec="$(git rev-parse --git-common-dir)/ship/<branch>"
mkdir -p "$(dirname "$rec")" && printf 'head %s\n' "$(git rev-parse HEAD)" > "$rec"
```
The record is a file. A branch name with `/` (`feat/x`) makes folders above it, so create only its parent folder. Why: `mkdir -p` on the record's own path makes a folder the hook can't read.
A fix made during the interview is committed before the record is written.
The user may skip the interview: write the record and say in the reply that it was skipped. A skip covers that interview only; after new commits, ask again about the short re-interview. Never skip it unasked, and never call it compulsory. Why: it teaches and never blocks, but the choice is the user's.

### 5. Push and open the PR or MR
- The commit being pushed needs a passing `prove` receipt. Run `prove` if it has none, and again after any commit made in steps 2 to 4. Why: the hook blocks a push whose exact commit wasn't proved.
- Show the exact commands and the description; run them after a yes:
  - GitHub: `git push -u origin <branch>`, then `gh pr create --base <default> --title "<title>" --body-file <file>`
  - GitLab: `git push -u origin <branch>`, then `glab mr create --target-branch <default> --title "<title>" --description "<text>" --yes`. Never `--fill` or `--push`. Why: both push without the shown command.
- Write the title and description with the `writing-pr` skill. The verification results go in chat, not in the description.
- Stop. Never merge, approve, or push to the default branch.

### 6. When the hook blocks a push
New commits since the interview: run a short interview on only those commits, then update the record and push again. A `prove:` block: run `prove` on that commit and push again. `SHIP_SKIP=1` and `PROVE_SKIP=1` are the user's call for one push; never suggest them yourself.

### 7. After the merge
When the user says it is merged, list what to clean up: the local branch, worktrees and review copies for it, `.prototype/` folders for it, and `.git/ship/<branch>`. Show each before deleting; delete after a yes. Why: a worktree can hold uncommitted work.

## Example

User: "Ship it." Branch `fix/retry-dupes`, 3 files changed.
1. `review`: clean. Done gate: all done; docs not applicable (no visible change). `bar-guard`: one hit, `t.Skip` in `retry_test.go`. Asked; the user says remove it; removed, tests pass.
2. Commit plan: 1 commit, "fix: make webhook retries idempotent". Yes, committed.
3. Interview: 7 questions, 5 right, 1 partly, 1 wrong (the retry cap is 3 in `retry.go:41`, not 5). Record written.
4. No prove receipt for the commit: ran `prove`. Retry flow PROVED by a subagent.
5. Showed `git push -u origin fix/retry-dupes` and the PR text. Yes: pushed, PR opened. Stopped.

## Replies

Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

## References

- [references/interview.md](references/interview.md) - read before the interview, and for a short re-interview after new commits.
- [references/done.md](references/done.md) - the done gate, and what to do with bar-guard hits.
- [references/hook.md](references/hook.md) - read at step 1, or when the hook blocks.
- [references/deploy.md](references/deploy.md) - read only when the user says the change is being deployed.
- [references/release.md](references/release.md) - read only for a version bump, a tag or a changelog.
