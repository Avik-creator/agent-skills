# The pre-push hook

Read before installing the hook (SKILL.md step 1) or when it blocks a push.

## What it does
On every push of a branch, `scripts/pre-push` reads the record at `<git-common-dir>/ship/<branch>` (`head <sha>`) and decides:

| State | Result |
|---|---|
| No record for the branch | Blocked: no interview yet |
| The record matches the commit being pushed | Pushed |
| New commits since the recorded one | Blocked, and lists them; run the short re-interview |
| Rebased or amended, but the same change against the default branch (same `git patch-id`) | Pushed, and the record moves to the new commit |
| Rewritten, and the change differs | Blocked |
| Tags, and deleting a branch | Not checked |

`SHIP_SKIP=1 git push` skips the check for one push. That is the user's choice; the agent never suggests it. The record and the hook live inside `.git/`, so nothing is committed and teammates are not affected.

## Install (after a yes)
```bash
<skill dir>/scripts/install-hook <skill dir>/scripts/pre-push
```
- It copies the check to `.git/ship/pre-push` and writes a small `pre-push` hook that runs it.
- An existing `pre-push` hook is moved to `.git/ship/pre-push.previous`. It still runs after the check, with the same input.
- Running it again refreshes the check; it is safe to repeat.
- Worktrees share it, because it uses the common git dir.
- Exit 3: `core.hooksPath` is set (husky, lefthook or another manager owns the hooks). Nothing was changed. See below.

## When a hook manager owns the hooks
The manager's hook files are usually committed, so changing them changes every teammate's setup. Ask first. If the user agrees, add this at the top of that `pre-push` file. It does nothing for anyone without the local check installed:
```sh
f="$(git rev-parse --git-common-dir)/ship/pre-push"
if [ -x "$f" ]; then input=$(cat); printf '%s\n' "$input" | "$f" "$@" || exit 1; fi
```
Then copy `scripts/pre-push` to `.git/ship/pre-push` and make it executable. If later steps in that file read the push list from stdin, they need `printf '%s\n' "$input" |` in front of them, because the line above has already read it.

## Remove
Delete `.git/hooks/pre-push`, move `.git/ship/pre-push.previous` back to `.git/hooks/pre-push` if it exists, then delete `.git/ship/`. Show these commands and run them after a yes.
