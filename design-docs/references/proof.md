# Proof: how each goal will be checked

Read when a plan, PRD or spec has goals or requirements. Settle the proof in the discussion, with the user, before any code. Why: the agent that builds the work must not decide what counts as right, or it will meet a bar it set itself.

## What every goal gets

For each goal, requirement and changed behaviour, settle four things with the user:

- **Surface:** where a real user or program meets it: a page, an API response, a CLI's output and exit code, a row in the database, an email, a message. Check that surface, not the one that is easiest to test. Why: a fully mocked test proves no goal.
- **Answer key:** what right looks like, written or chosen by someone other than the builder. See the ladder below.
- **Check:** how the result is compared with the answer key: a script, a scripted user flow, or a fresh agent that never saw the code and gets only the goal.
- **Pass signal:** the exact result that counts as a pass, such as "status 201 and one new row", or "pixel difference under 3% at 375, 768 and 1440 wide".

The building agent reading its own diff is not a check.

## The answer-key ladder

Pick the highest rung that exists, and say which rung it is.

1. **An old system** that does the same job. Drive both and compare.
2. **A snapshot:** a saved good result, such as screenshots, API responses or CLI output from before the change, or an approved prototype.
3. **An independent test suite** that does not depend on the code it checks.
4. **A written spec:** steps and acceptance criteria, specific enough to fail.
5. **Nothing:** the agent decides what right means. Flag it as the weakest, and ask whether a higher rung can be made.

Which rung fits which change:
- **Bug fix:** a reproduction that fails before the fix and passes after.
- **Refactor or migration:** the app before the change (rung 1 or 2). Nothing should change.
- **New screen or flow:** the approved prototype or design (rung 2), plus spec lines for behaviour a picture can't show.
- **New surface that copies an existing one** (an MCP server beside an API): parity with the existing one. When one change edits both sides, add spec lines too, because both sides can be wrong the same way.
- **Every change:** the existing flows it touched, directly or through shared code (schema, auth, navigation, a shared component), checked against the app before the change.

## Pixel thresholds for screens

- **Refactor or migration:** 0%. Any difference fails, and the pixel delta is investigated.
- **New screen against a prototype or design:** the user picks the threshold. Propose 3% at 3 widths.
- Freeze what moves: fonts, data, time, animations and the browser version. Why: without that, the number is noise.

## Rules

- Mark each check machine-run or human-judgment. For a human-judgment check, propose one that could replace it. A human-only check stays only if the user confirms it, and it never counts as proof.
- If a check needs an environment that doesn't exist yet (a test account, a local email catcher, seed data), ask whether to set it up first. Never swap in a weaker check quietly.
- Answer keys use made-up test data only. Never real names, emails or records.
- Changing an answer key later is the user's decision, never the builder's.

## In the document

Add a **Proof** section: one row per check.

| # | Proves | Surface | Answer key (rung) | Check | Pass signal |
|---|---|---|---|---|---|
| 1 | Goal 1: signed-out user gets a reset link | email captured locally | spec (4) | scripted flow | one email within 1 minute, link opens the reset page |
| 2 | Existing sign-in still works | sign-in page | app before the change (2) | browser drive | same screens and same rows as before |

Every check must pass before the work counts as done.
