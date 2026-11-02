# Finding the commit with git bisect

Read before bisecting. Use it when the bug is present now and a known commit or release was clean.

## Before starting
- The working tree must be clean. If there are uncommitted changes, stop and ask, because checking out commits either carries them along or fails.
- You need a check that tells the bug apart from every other failure. Run one targeted check, never the whole suite. Why: an unrelated failure at an old commit marks it bad and sends the search down the wrong half.
- Keep the check and the step script **outside the repo**. Why: a file inside the repo changes or disappears as bisect checks out older commits.

## Exit codes
From `git help bisect`. Checked against git 2.54.

| Exit code | Meaning |
|---|---|
| 0 | good: the bug is absent |
| 1–127, except 125 | bad: the bug is present |
| 125 | skip: this commit can't be tested |
| anything else | bisect stops |

126 and 127 count as bad. A missing command or a script that can't run (127, "command not found") therefore marks a good commit bad. The step below turns everything except a clean 0 or 1 into a skip.

## The step script
The check exits 0 when the result is right and 1 when it is wrong. It exits 2 when it can't judge, for example when the function is missing at that commit.

```sh
#!/bin/sh
# One bisect step: 0 good, 1 bad, 125 skip. Lives outside the repo.
BUILD='<the repo install and build command>'
CHECK='<command that runs the check, by absolute path>'
sh -c "$BUILD" >/dev/null 2>&1 || exit 125
sh -c "$CHECK"; first=$?
sh -c "$CHECK"; second=$?
[ "$first" = "$second" ] || exit 125
[ "$first" -le 1 ] || exit 125
exit "$first"
```

- A build failure exits 125. Why: an unbuildable commit says nothing about the bug.
- The check runs twice, and a result that flips is skipped rather than blamed.
- Pin time, randomness, network and time zone inside the check, as reproduce.md says.

Tested on a scratch repo with eight commits: a good commit gave 0, a bad one 1, and an unbuildable commit, a flaky result and a check that couldn't judge each gave 125. Bisect named the right commit.

## Running it
```sh
git bisect start
git bisect bad <bad commit>
git bisect good <good commit>
git bisect run /abs/path/step.sh
git bisect log        # copy this into the report before resetting
git bisect reset      # back to the original HEAD
```

- A range known to be unbuildable can be skipped up front: `git bisect skip <from>..<to>`.
- If bisect ends with only skipped commits left, it lists the candidates instead of one commit. Report the list; don't pick one.
- The first bad commit is where the behaviour changed. It may have exposed an older bug rather than caused it, so read its diff and trace to the source (SKILL.md step 6).
