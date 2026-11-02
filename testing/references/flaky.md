# Flaky and newly red tests

Read when a test passes and fails on the same code, or a UI test turned red.

## Is it flaky?
Flaky means different results on the same commit. One red run after a change is a failure until shown otherwise. Why: calling a real regression "flaky" hides it.

## Reproduce before fixing
Run it many times until it fails at least once, then run it the same way after the fix and expect no failures:
- Playwright: `--repeat-each=<n>`, with more workers to add pressure.
- Go: `go test -run <name> -count=<n> -race`.
- Other runners: a loop in the shell.
Pick the count with the user; say what you ran. Without a failing reproduction you can't tell a fix from luck.

## Find the cause, in this order
1. **Passes alone, fails in the suite**: shared state or order. Look for data another test leaves, a shared account, a global, a fixed id.
2. **Fails more in CI**: timing or environment. Look for a missing wait on an async step, CPU contention, a different time zone, locale or image.
3. **Fails at certain times**: dates. Look for "now", midnight, month end, DST. Freeze the clock.
4. **Makes an external call**: that service. Stub it at the HTTP boundary.
5. **Otherwise**: an async step nobody waits for. Wait on the result the user sees.

## A UI test that turned red
Find out why before editing the test. A locator timeout can be an app regression, a login or network problem, or a page that isn't ready. Read the trace first. After fixing a locator, run the whole affected file, because the new locator may match the wrong element. Confirm a feature was really removed before deleting its tests.

## Not fixes
- A retry, a sleep, or a longer timeout. Each one hides the race. If the config retries, run with `--fail-on-flaky-tests` (Playwright) so a pass after a retry still fails.
- Skipping the test. A skip is flagged by ship's bar-guard; it needs the user's yes, a reason in the code, and an issue link.
