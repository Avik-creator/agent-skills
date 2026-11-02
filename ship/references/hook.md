# The pre-push hook

Read at SKILL.md step 1, or when the hook blocks a push.

## What it does
On every push of a branch that `deliver` marked (a file at `<git-common-dir>/deliver/<branch>`), `scripts/pre-push` reads the record at `<git-common-dir>/ship/<branch>` (`head <sha>`) and decides:

| State | Result |
|---|---|
| No record for the branch | Blocked: no interview yet |
| The record matches the commit being pushed | Pushed |
| New commits since the recorded one | Blocked, and lists them; run the short re-interview |
| Rebased or amended, but the same change against the default branch (same `git patch-id`) | Pushed, and the record moves to the new commit |
| Rewritten, and the change differs | Blocked |
| Tags, deleting a branch, and any branch deliver did not mark | Not checked |

`SHIP_SKIP=1 git push` skips the check for one push. That is the user's choice; the agent never suggests it. The record and the hook live inside `.git/`, so nothing is committed and teammates are not affected.

## The global hooks
The library's `_hooks/` folder holds one `pre-push` for every repo on the machine. On branches deliver marked it runs this check and prove's; on every push it then runs the repo's own `pre-push` hook. Its other files pass every other hook (`pre-commit`, `commit-msg` and the rest) to the repo's own hook, so no repo loses a hook.
```bash
<skill dir>/../_hooks/install          # turn on: sets git's global core.hooksPath to that folder; repeating it changes nothing
<skill dir>/../_hooks/install check    # is it active in this repo?
```
- Run `install` at the start of every ship run, without asking: it changes only the global git setting, and it checks only branches deliver marked, so other pushes are untouched. Tell the user in one line when it turned the hooks on.
- Exit 3: the global `core.hooksPath` already points somewhere else. Nothing was changed. Ask the user which one should win.
- `install check` fails with "this repo sets its own core.hooksPath": a hook manager (husky, lefthook) owns this repo's hooks, and the global hooks don't run here. See below.
- An older per-repo install (a `pre-push` with `ship-wrapper` in it) makes the check run twice. Remove it as below.

## When a hook manager owns the hooks
The manager's hook files are usually committed, so changing them changes every teammate's setup. Ask first. If the user agrees, add this at the top of that `pre-push` file. It does nothing for anyone without the global hooks:
```sh
g=$(git config --global --get core.hooksPath)
if [ -n "$g" ] && [ -x "$g/pre-push" ]; then input=$(cat); printf '%s\n' "$input" | "$g/pre-push" "$@" || exit 1; fi
```
If later steps in that file read the push list from stdin, they need `printf '%s\n' "$input" |` in front of them, because the line above has already read it.

## Remove
- The global hooks: `git config --global --unset core.hooksPath`.
- An older per-repo install: delete `.git/hooks/pre-push`, and move `.git/ship/pre-push.previous` back to `.git/hooks/pre-push` if it exists. Keep `.git/ship/`, which holds the interview records.

Show these commands and run them after a yes.
