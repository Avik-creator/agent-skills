---
name: deliver
description: Takes a piece of work from request to an open PR with proof. It asks everything up front - what the user wants, and for each goal where it is seen, what right looks like and what counts as a pass - prototypes screens and logic first, plays the whole plan back for one confirmation, locks the answer keys, then builds, proves, reviews and opens the PR without further questions unless something blocks it. Works with any coding agent. Use when the user says deliver this, build this end to end, take this from idea to PR, do the whole thing, "ask me everything first, then go", or hands over a feature, screen, flow, endpoint or change they want built and proved. Not for a quick question or a one-line fix - use coding; not for only discussing or writing a design doc - use design-docs; not for checking a change that already exists - use prove.
---

# Deliver

All the user's decisions happen before any code, including how the work will be proved. After one confirmation, the agent builds to answer keys it did not write, and stops only when it is blocked.

## Quick start
1. **Ask:** what is wanted, and how each goal is proved.
2. **Prototype** screens and logic, unless the work doesn't need one.
3. **Play back** the whole plan in one message; after a yes, lock the keys.
4. **Build** in small steps, each one checked.
5. **Prove** it with `prove`.
6. **Review** it with `review`.
7. **Open the PR** with `ship`, then stop.

## 1. Ask
- Read the repo first: code, routes, docs, configs, the test tools that exist. Ask only what the repo can't answer. Why: the user's time goes on decisions, not facts.
- Run the discussion as `design-docs` does in its `interview.md`: one decision at a time, the highest-risk first, each with a recommended answer.
- For every goal, settle its proof as `design-docs` does in its `proof.md`: surface, answer key and its rung, check, and pass signal.
- Settle the stop-and-ask list now: which schema, API, security, user-visible or destructive changes the user allows. Anything outside it stops the build later.
- A bug fix needs less: the reproduction is the answer key. Confirm the expected behaviour, and skip to the playback.

## 2. Prototype
Inside this workflow a prototype is the default, decided by this rule:
- **Prototype:** any screen or user flow, and any state model or business rule with more than two states.
- **Skip,** with the reason said: a bug fix, a refactor, a config change, or backend plumbing whose shape is obvious.

Name the choice in one line ("prototype: UI, the question is X" or "skip: bug fix"); the user can overrule it. Build it with the `prototype` skill. When the user approves it, it becomes an answer key: screenshots of the chosen variant at the plan's widths, or the scenarios and their expected results. Why: a prototype the user clicked through says what right looks like better than any description.

## 3. Play back
Send one message with everything:
- the goals and non-goals, each decision with its one-line reason
- the Proof table: one row per check, with surface, key and rung, check, and pass signal
- the prototype verdict, or why there was none
- the stop-and-ask list, and the choices left to the agent (names, file layout, test framework)
- numbered defaults the user can veto, such as "pixel threshold 3% at 375, 768, 1440"

Write nothing until the user confirms. A hedged answer ("I guess", "probably") is not a yes. Then save the keys in `.prove/keys/<task>/` (`plan.md` plus snapshots) and run prove's `scripts/keys lock`. Why: a locked key can't be lowered quietly during the build.

## 4. Build
- Work on a branch, never the default branch. Follow `coding`, with `go`, `database` or `security` alongside where `which-skill` says.
- Split the work into small steps that each end in a state you can check. Run that check before the next step. Why: a break found at the step that caused it is cheap to find.
- Order commits so they prove themselves: for a bug, the failing test first and the fix on top.
- Never edit an answer key or the verifier. If a key looks wrong, stop and ask.
- Write down each judgment call (a name, a trade-off, an edge case handled) for the final report.

## 5. Prove
Run `prove` on the committed head. It sets up the repo's verifier first if there is none. A BROKEN flow goes back to the build: at most 3 rounds, then stop and report. BLOCKED or UNPROVEN means stop and tell the user what is missing, never a weaker check.

## 6. Review
Run `review` on the branch. If its fixes change code, commit them and run `prove` again. Why: the push gate needs a receipt for the exact commit being pushed.

## 7. Open the PR
Run `ship`: its done gate, commit plan, interview and push, each after a yes. For a visual change, the before and after screenshots from the prove run go in the PR's before-and-after table, after a yes. Stop at the open PR; the user merges.

## When to stop and ask
Only when: a decision isn't in the playback, a change is outside the stop-and-ask list, a key and the app disagree, a check is BLOCKED, or 3 fix rounds didn't pass. Ask one question with a recommended answer, then carry on.

## Replies
Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

During steps 1 to 3, each reply moves one decision forward and ends with the next question. At the end, report: the PR link, each flow's verdict with its key and evidence path, the judgment calls made, and what wasn't covered and why.
