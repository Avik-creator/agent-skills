---
name: deliver
description: Takes a piece of work from request to an open PR with proof. It asks everything up front - what the user wants, and for each goal where it is seen, what right looks like and what counts as a pass - prototypes screens and logic first, plays the whole plan back for one confirmation, locks the answer keys, then builds, tests, proves, reviews and opens the PR, with no questions between the playback and the final ship checks unless something blocks it. Works with any coding agent. Use when the user says deliver this, build this end to end, take this from idea to PR, do the whole thing, "ask me everything first, then go", or hands over a feature, screen, flow, endpoint or change they want built and proved. Not for a quick question or a one-line fix - use coding; not for only discussing or writing a design doc - use design-docs; not for checking a change that already exists - use prove.
---

# Deliver

All of the user's decisions happen before any code, including how the work will be proved. After one confirmation, the agent builds to answer keys it did not write, and asks again only when blocked.

## Quick start
Each step calls these skills, in this order. Why: the user should know what runs, what it hands on, and when it will stop.

| Step | Skills called | What happens | Hands on | Stops for the user when |
|---|---|---|---|---|
| 1. Ask | `design-docs` (interview.md, proof.md); `prove` (setup.md) when the repo has no verifier | Reads the repo, asks one decision at a time, settles each goal's proof | The decided list and the Proof table | Each question (this step is the asking) |
| 2. Prototype | `prototype` | Builds screen variants or a clickable logic demo, unless the rule in step 2 skips it | The chosen variant and its scenarios | The user picks a variant |
| 3. Play back | `prove` (`scripts/init`, `scripts/keys`) | One message with the whole plan; after a yes, creates the branch if step 2 didn't, saves and locks the keys | Locked keys in `.prove/keys/<task>/` | The user confirms |
| 4. Build and test | `coding` and `testing`; `go`, `database`, `security`, `observability` alongside where `which-skill` says; `prove` (methods.md, checker.md) for each step's check | Small steps, each one a build, check, fix loop until a locked script or a fresh agent passes it; committed in order | Commits on the branch | A change outside the stop-and-ask list, a key looks wrong, or a step still fails after 3 fixes |
| 5. Prove | `prove`, with `automation` to drive a browser | Drives every touched flow, a fresh agent gives the verdict, writes the receipt | A PROVED receipt for the commit | BLOCKED, UNPROVEN, or the fix rounds run out |
| 6. Review | `review` | A fresh reviewer proves each finding; fixes are committed, then `prove` runs again | A clean review and a new receipt | A fix needs a schema, API, security or user-visible change |
| 7. Open the PR | `ship`, with `writing-pr` for the description | Done gate, bar-guard, commit plan, interview, push, PR | The open PR | Always: the commit plan and the push each need a yes; the interview runs unless the user skips it |

The user merges. Nothing in this workflow merges, deploys or pushes to the default branch.

## 1. Ask
- First turn on the global hooks, as `prove` step 1 says. Why: every later push is checked, with no repo left out.
- Read the repo first: code, routes, docs, configs, the test tools that exist. Ask only what the repo can't answer. Why: the user's time goes on decisions, not facts.
- Read the data the feature runs on too, from the repo or a local copy (content files, fixtures, a local database): count it, and look for duplicates, case or spelling variants, and outliers. Never a production or shared system, and never copy real records into keys or prototypes. Why: the data often decides the design (200 categories make a different screen from 10), and the code doesn't show it.
- Run the discussion as `design-docs` does in its `interview.md`: one decision at a time, the highest-risk first, each with a recommended answer. Why: a decision asked before the one it depends on gets undone.
- For every goal, settle its proof as `design-docs` does in its `proof.md`: surface, answer key and its rung, check, and pass signal.
- No `.prove/verifier.md` in this repo: the method choice from prove's `setup.md` is one of the questions. Why: setting it up mid-build would mean asking then.
- Settle the stop-and-ask list now: which schema, API, security, user-visible or destructive changes the user allows. Why: these are expensive to undo, and anything outside the list stops the build.
- A bug fix needs less: the reproduction is the answer key. Confirm the expected behaviour, then go to the playback. Why: the bug already defines right.

## 2. Prototype
Inside this workflow a prototype is the default, decided by this rule:
- **Prototype:** any screen or user flow, and any state model or business rule with more than two states.
- **Skip,** with the reason said: a bug fix, a refactor, a config change, or backend plumbing whose shape is obvious.

Name the choice in one line ("prototype: UI, the question is X" or "skip: bug fix"); the user can overrule it. Name the work branch in the prototype's one-line plan; after its yes, create the branch, then build it with the `prototype` skill. Why: the host-page wiring is an edit to a real file, and it must never sit on the default branch. For a UI prototype, start the app as a process that outlives your turn, open every link yourself before sending it, send the start command with the links, and keep it running until the user picks. Why: a link that is dead when the user clicks it wastes a turn. The approved prototype becomes an answer key for structure and behaviour: which parts are on the page, in what order, where each step leads, and the scenarios with their expected results. It is rough on purpose, so pixel checks use a design export or the base commit, never the prototype. Why: clicking through a real draft settles what right looks like better than any description.

## 3. Play back
Send one message with everything:
- the goals and non-goals, each decision with its one-line reason
- the Proof table: one row per check, with surface, key and rung, check, and pass signal
- the prototype verdict, or why there was none, and any flow left not covered on purpose
- the stop-and-ask list, and the choices left to the agent (names, file layout, test framework)
- who judges build steps in step 4: a subagent or another agent's CLI (`cli:<name>`). If neither runs here, step 4 judges with scripts only and the fresh-agent check waits for step 5; say so. Why: the build loop must never wait on the user
- numbered defaults the user can veto, such as "pixel threshold 3% at 375, 768, 1440"

Write nothing until the user confirms. A hedged answer ("I guess", "probably") is not a yes. The yes covers the verifier, and the branch if step 2 didn't create it. Then, in this order, so the bar is set and tested before any code:
1. Run prove's `scripts/init`, write the verifier if it is new, and save the keys in `.prove/keys/<task>/`: `plan.md`, base-commit and design snapshots, the prototype notes, locked checks, and the screenshot script with its freeze step. Why: step 4 must capture exactly as the keys did, or every diff is noise.
2. A fresh agent opens every snapshot and confirms it shows what it claims: the right page and width, nothing blank, nothing mid-animation. A bad base-commit snapshot is retaken and checked again; a bad design export goes to the user. Why: a blank key passes against a blank build.
3. Prove the checks can fail: a new verifier says BROKEN on a broken copy (prove's setup.md step 5); a check for a new flow fails on the base commit; a check of what must stay the same fails on a broken copy. Why: a check never seen failing turns the loop green for free.
4. Run `scripts/keys lock <task>`. Why: a key saved after the lock blocks the run, and a locked key can't be lowered quietly.

If the snapshot check (item 2) or the failing proof (item 3) still fails after one retry, or a fix would change a method the user chose, stop and ask.

## 4. Build and test
- Work on a branch, never the default branch. Follow `coding`, with `go`, `database` or `security` alongside where `which-skill` says, and write the tests with `testing`. Why: those skills own the rules for each kind of code.
- **Every step runs as a loop, with no user in it: build, check, fix, check again.** Split the work into small steps that each end in a state you can check. Why: a break found at the step that caused it is cheap to find.
  - Before building a step, name the Proof-table rows (or parts of a row) it finishes. Why: the judge grades against the user's rows, never a goal the builder wrote.
  - Build it, then run the checks for those rows, plus every check of what must stay the same (the tests so far, the 0% screenshots of unchanged parts). Checks for rows not yet reached don't count. A screen step (anything that changes what a screen shows, shared CSS and components included) adds screenshots with the locked screenshot script and the pixel compare where the row has a pixel key. A step that adds a reproduction test passes when that test fails on the unfixed code for the bug's reason.
  - A failing script is the FAIL note. When every script passes, a new fresh agent judges, unless locked scripts fully cover the rows (never a pixel diff alone). The agent gets only the rows, their keys and the evidence (output, responses, screenshots, diff image), never the fix notes, and returns PASS or FAIL with exactly what is wrong. Why: a judge that sees the fix notes grades the intent. A step that finishes no row is checked by the build and the tests, and judged with the next step that does.
  - On FAIL, fix from the note and check again; at most 3 fixes after the first FAIL, matching prove, then stop and ask. Why: the builder never judges its own work, and a passing number alone can't tell a blank page or an empty list from a right one.
- Order commits so they prove themselves: for a bug, the failing test first and the fix on top. Why: a reviewer sees it fail, then pass.
- Never edit an answer key. `verifier.md` changes only through prove's drift rule, reported to the user. If a key looks wrong, stop and ask. Why: the builder must never set its own bar.
- Write down each judgment call (a name, a trade-off, an edge case handled) for the final report.

## 5. Prove
Run `prove` on the committed head. Its rules decide the fix rounds for a BROKEN flow. BLOCKED or UNPROVEN means stop and tell the user what is missing. Why: a weaker check that passes reads as proof.

## 6. Review
Run `review` on the branch. If its fixes change code, make all of them and commit them, then run `prove` once on the new head. Why: the push gate needs a receipt for the exact commit being pushed.

## 7. Open the PR
Run `ship`. Its commit plan, interview and push are the last questions, kept on purpose. The user may skip the interview, as `ship` says; the agent never skips it unasked. Any commit made inside `ship` (a bar-guard fix, an interview fix) means `prove` runs again before the push. For a visual change, the before and after screenshots from the prove run go in the PR's before-and-after table, after a yes. Stop at the open PR; the user merges.

## When to stop and ask
Only when: a decision isn't in the playback, a change is outside the stop-and-ask list, a key and the app disagree, a check is BLOCKED or UNPROVEN, the fix rounds ran out (prove's, or a build step's), or a snapshot or the failing proof still fails before the lock. Ask one question with a recommended answer, then carry on.

## Example
User: "Deliver a CSV export on the orders page."
1. Asked 6 questions, one at a time: columns, filters, file size limit, who may export, the proof for each goal, and the methods for this repo's first verifier.
2. Prototype: UI, "where does the export button go?". Variant B chosen.
3. Playback confirmed. Keys locked as `orders-csv`: spec rows, variant B's structure, base-commit screenshots of the orders page.
4. Built in 4 commits, with tests. `prove`: export flow PROVED (subagent), orders page 0% difference. `review`: one fix, so proved again. `ship`: interview passed, PR opened. Stopped.

## Replies
Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

During steps 1 to 3, each reply moves one decision forward and ends with the next question. At the end, report: the PR link, each flow's verdict with its key and evidence path, the judgment calls made, and what wasn't covered and why.
