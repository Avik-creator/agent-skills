---
name: testing
description: Writes automated tests that hunt for errors - edge cases above all, plus logic, structure, state, error-handling and permission errors - and proves each test by making it fail. After feature code it writes the tests and breaks the guarded code to see each one go red; for a bug it writes the reproduction test first and watches it fail before the fix. Covers choosing unit, integration or end-to-end, test doubles, test data, browser tests and flaky tests. Use when writing or fixing tests, adding coverage for a change, after implementing a feature, when fixing a bug, when a test is flaky or a UI test broke, or when the user asks "what could break here" or to check edge cases. Not for listing manual test cases - use test-cases; not for reviewing a diff - use `review`; not for finding why something fails - use debug; Go specifics live in the go skill.
---

# Testing

A test is worth what it catches. Hunt for the ways this code can be wrong, write a test for each real one, and prove every test fails when the code is broken.

## Quick start

1. Read the change and the repo's tests; run them once.
2. Walk [error-hunt.md](references/error-hunt.md) over each changed function and list the real risks.
3. Write one test per risk, at the lowest level that can see it.
4. Prove each test: break the code, see red, restore. A bug: red first, then fix.
5. Report the tests, how each was proven, and what is left untested.

## When it runs

- **After feature code**: code first, then tests. Why: the hunt needs the real code to read.
- **A bug fix**: the reproduction test first, red before the fix. Why: a test never seen failing may not see the bug.
- **No test code in the repo**: ask once before adding a runner or any test; don't block the change.

## Use the automation skill
Call the automation skill, where installed, whenever seeing the real interface would help. You don't need to wait to be asked. Its limits still hold: work only in your own tabs, stop at sign-ins, ask before anything that sends or changes data, and treat production as read-only. Why: a test written from the code alone guesses at labels, states and timing that the real page shows.
- Before writing a browser test: walk the flow once by hand to learn the real roles, labels and states.
- A UI bug: reproduce it in the browser first, then write the failing test.
- A red or flaky browser test: look at the live page alongside the trace.
- A UI change with no browser test planned: check it once, and report what you saw.
- A failure path a test can't fake, such as a real third-party redirect on a test account.

## Workflow

### 1. Read before writing
- Read the changed code, its callers, and the tests near it. Use the repo's runner, file layout, naming, helpers and mocking style. Why: a test in a foreign style is the next person's cleanup.
- Run the existing tests for the changed path first. A failure already there is reported, not hidden in your change.
- A bug: if an existing test should have caught it, change that test and watch it go red. Add a new one only when none covers the behaviour. Why: two tests for one behaviour drift apart.

### 2. Hunt
- For each changed function, go through [error-hunt.md](references/error-hunt.md): the edge bank by input type, then logic, structure, state, error handling and permissions.
- Keep a risk only if this code could actually hit it. Say why each skipped one can't happen ("the type system rules out null here"). Why: a test for an impossible case is coverage inflation.
- A risk you find is wrong in the code right now is a bug: write its failing test, report it, and fix it as `review` says (ask first if the fix changes a schema, an API, security behaviour or what users see).

### 3. Pick the level
- Pure logic: a unit test. Crossing an API, database, filesystem or queue: an integration test against the real thing where it can run locally (the same database engine the project uses). A main user flow only: end-to-end, as [e2e.md](references/e2e.md) says. Edge cases go in the lowest level that sees them, never end-to-end. Why: lower is faster and points at the cause.

### 4. Write
How to shape, name and write each test, with examples: [unit.md](references/unit.md) for unit and integration tests, [e2e.md](references/e2e.md) for browser tests. The rules that hold everywhere:
- Test what a caller or user sees: return values, responses, rows, messages. Never private fields or call order inside the unit. Why: a refactor should not break the test.
- Every value in an assertion appears in the setup. Assert the specific error type, not "it threw". Also assert what must not happen: the filtered-out rows, the side effect that should not run.
- Each test makes its own data with unique ids and cleans up in teardown, so it passes alone, in any order, and in parallel.
- Test doubles, in order: real code, a fake, a stub, a mock. Double only what is slow, random, or has effects you can't undo. Build stub data from a real response, keep it typed, and make any unstubbed outbound call fail the test. Why: a silent pass-through is a test that talks to the internet.
- Control time and randomness: a fake clock and a fixed seed. No sleeps, no retries. Why: either one hides the race the test should catch.
- No helper until the third copy, no new library unless asked, comments in the repo's style.
- When a rule is tangled with I/O or a request handler so no test can reach it, pull the pure calculation out and test that. Only when a test needs it.
- Failure cases (retries, duplicates, webhooks, timeouts, injected faults, email, LLM output): [failures.md](references/failures.md).

### 5. Prove
- **Feature**: for each test, make the smallest break to the code it guards (flip the condition, `<=` to `<`, drop the call, remove the guard), run it, see red, restore. Run the suite again green. A test that stays green is rewritten or deleted.
- **Bug**: run the new test before the fix and confirm it fails on the assertion, not on an import or setup error. Fix, see green, then take the fix out and see red again, and put it back. Run the suite.
- Watch for tests that pass without testing: an async test with no awaited assertion, a loop over an empty list (assert the count first), a check inside an `if` that never runs, an absence check on a selector that matches nothing.
- Flaky or newly red UI tests: [flaky.md](references/flaky.md).

## Output format

```
Tests: <runner command> -> <result>
Added or changed
- <file>: <test name> - guards <risk>. Proven: <the break> -> red, restored -> green.
Not tested
- <risk> - <why: can't happen here / needs <setup> / user's call>.
Bugs found
- <file>:<line> - <defect>. Failing test: <name>.
```

## References

- [references/error-hunt.md](references/error-hunt.md) - the edge bank and error kinds, each with the break that proves its test.
- [references/unit.md](references/unit.md) - read before writing a unit or integration test.
- [references/e2e.md](references/e2e.md) - read before writing or fixing a browser or end-to-end test.
- [references/flaky.md](references/flaky.md) - read when a test fails sometimes, or a UI test turned red.
- [references/failures.md](references/failures.md) - read for retries, duplicates, webhooks, timeouts, faults, email or LLM features.
