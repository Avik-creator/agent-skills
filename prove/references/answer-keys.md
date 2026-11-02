# Answer keys

Read when choosing or saving an answer key. An answer key says what right looks like. The agent that made the change never writes it alone, and never edits it.

## The ladder, best first
1. **An old system** that does the same job: drive both and compare outputs and stored state.
2. **A snapshot:** a saved good result: screenshots, API responses, CLI output, generated files, or a design export.
3. **An independent test suite** that does not depend on the code it checks.
4. **A written spec:** steps and acceptance criteria, specific enough to fail. Usually the Proof section from `design-docs`.
5. **Nothing:** the agent decides what right means. The verdict is UNPROVEN, and the report proposes a higher rung.

## Which key for which flow
- **A new flow:** the user's spec, plus the approved prototype for its structure: which parts are on the page, in what order, and what each step leads to. A prototype is rough on purpose, so it is never a pixel key. Pixel-diff a new screen only against a real design export (a Figma frame) or the base commit. If it copies an existing surface (an MCP server that does what the API does), parity with that surface too: the same action through both leaves the same data. When the change edits both sides, add spec lines, because both can be wrong the same way.
- **An existing flow the change touched:** the app before the change: the base commit running beside it in a worktree under `/tmp` on another port (rung 1), or a snapshot of it (rung 2). The smoke test for a change of new files only uses the same key.
- **A bug fix:** the reproduction. It fails on the base commit and passes on the change.
- **A refactor:** the app before the change, and nothing may differ.
- **A migration** (the old system is deleted at the end): the old system, side by side, on the same inputs. Before it is deleted, save its results as snapshots.

## Saving keys
- Put each task's keys in `.prove/keys/<task>/`: the proof plan (`plan.md`, one row per check: proves, surface, key and rung, check, pass signal), plus snapshots and expected outputs.
- Take every snapshot before the lock, from a known good state: the base commit, a design export, or the prototype the user approved. Never from the change being checked. Why: a snapshot added after the lock changes the keys, and the run stops as BLOCKED.
- Scripted checks that should count as the checker go in `keys/<task>/checks/`, before the lock.
- Write down the differences that are allowed (new ids, timestamps, a bug the change fixes) next to the snapshot, and keep the list short. Why: a comparison that ignores too much hides real breaks.
- Made-up test data only. Never real names, emails or records.
- After the user approves, run `scripts/keys lock <task>`. Each task has its own lock, so approving one task never approves an edit to another.

## Screens and pixel diffs
- Compare screenshots at the widths the plan names, by default 375, 768 and 1440.
- **Refactor or migration:** a threshold of 0%. Any difference fails; look at the pixels that changed.
- **New screen against a design export:** the threshold the user chose, 3% by default.
- Freeze fonts, data, the clock, animations and the browser version, or the number is noise.
- A pass at one width says nothing about the others.

## When the app and a key disagree
Find out which is wrong from the commit that changed the behaviour. Report it to the user with the evidence. Only the user changes a key, and the task is locked again after their yes.
