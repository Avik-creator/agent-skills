# Reviewer brief

Read before briefing the reviewer (SKILL.md step 3). It covers making the copy, the brief itself, and what the reviewer sends back.

## 1. Make the copy

Make one copy per reviewer, outside the repo. Why: two reviewers mutating the same files would break each other's runs.

| Reviewing | Make the copy |
|---|---|
| A branch or commit range | `git worktree add --detach /tmp/review-<id> <head>` |
| A GitHub PR | `git fetch origin pull/<n>/head`, then `git worktree add --detach /tmp/review-<id> FETCH_HEAD` (GitLab: `merge-requests/<n>/head`) |
| Your own uncommitted work | See below |

Your own uncommitted work, staged and unstaged, plus new files:

```bash
WT=/tmp/review-<id>
git worktree add --detach "$WT" HEAD
git diff HEAD --binary | git -C "$WT" apply
git ls-files --others --exclude-standard -z | rsync -a --from0 --files-from=- ./ "$WT"/
git -C "$WT" add -A
git -C "$WT" -c user.name=review -c user.email=review@localhost commit -qm "review snapshot"
```

- The snapshot commit sits on a detached head in the copy, not on any branch. Why: the reviewer can now restore any file with `git checkout -- <file>` without losing the uncommitted work.
- If the tests need installed dependencies, install them in the copy from the lockfile (`npm ci`, `uv sync`, `go mod download`). For a PR from a fork or from outside the team, this is part of what the user approves. Why: installing runs the dependencies' scripts.
- Gitignored files such as `.env` are not copied. Add only what the tests need, and only local or test values.

## 2. The brief

Fill this in and send it as the whole prompt. Add nothing from your session.

```
You are reviewing a code change. Work only in <copy path>. Never edit <original repo path>.

The change: <git diff command>, commits: <git log --oneline output>.
It is <N> files, +<A>/-<D>.

What it should do:
- <intent bullet>
- <intent bullet>

Rules it should follow: <rule files, with paths>. Where they disagree with
<path to checks.md>, the rule files win.

How to work:
1. Read <path to checks.md>, <path to proof.md>, and the Signals section of <path to simplify.md>. On a strict review, also <path to strict.md>.
2. Run the tests and the linter in the copy. Report the commands and the results.
3. Review the diff against the intent and the rules, and read the code it calls and is called by.
4. Prove each finding as proof.md says. Anything you write goes in the copy.
5. Run only against local or test setups. No production, no real third-party services.

Send back, in this order, under 600 words:
- Ran: each command and its result.
- Findings: Blocker or Fix, [check], file:line, the defect in one line, and the proof:
  a command runnable from <copy path> and what it showed, or the rule quoted with its file:line.
  Add (pre-existing) when the proof also fails at the base.
- Couldn't prove: the risk, why it can't be proven here, how to check it.
- Judgement calls: file:line, the smell, one line each.
- Structure (strict review only): file:lines, what got worse, the simpler shape.
- Any test files you wrote, with their paths in the copy.
Zero findings is a valid result. Don't invent one.
```

## Two reviewers

Above about 400 changed lines, send two briefs in parallel, each with its own copy:

- **Intent reviewer**: the intent bullets, and the correctness, security and performance checks. Leave out the rule files and the smells.
- **Rules reviewer**: the rule files, and the architecture, readability and test checks plus the smells, and strict.md on a strict review. Leave out the intent bullets.

Why: each reviewer has less to hold, and one kind of finding can't crowd out the other.

## What comes back

- Check it against the format before using it. A finding with no proof goes to Couldn't prove.
- Keep the copy until you have re-run every proof (SKILL.md step 4).
