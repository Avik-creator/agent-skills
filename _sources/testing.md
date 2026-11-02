# Sources

Ideas were adapted and re-checked, not copied. Pulled 2026-11-02 from local downloads (no commit SHA recorded for the zips). Read by five subagents, each told to read every assigned file in full; their coverage reports said they did. The brief is summarised under "Read".

| Upstream | License | Taken |
|---|---|---|
| qa-skills-main `skills/test-reliability`, `test-data-management`, `test-suite-curation`, `test-migration`, `api-testing`, `service-virtualization`, `database-testing`, `contract-testing`, `ai-test-generation`, `unit-testing`, `coverage-analysis` | per that repo's LICENSE | The flake definition (different results on the same commit), reproduce before fixing, the triage order, no retry or sleep as a fix; ways a test passes without testing; tests own their data with unique ids and teardown cleanup; expected values in the server's canonical form; migration data survival; idempotency with the same key and a changed payload; webhook signature over raw bytes; a timeout test asserts the stub was reached; unstubbed calls fail the test; the negative oracle |
| qa-skills-main `skills/playwright-automation`, `selector-drift-recovery`, `agentic-browser-testing`, `accessibility-testing`, `cypress-automation`, `visual-testing`, `cross-browser-testing`, `mobile-testing` | same | Assert the right page before trusting a pass; absence checks pass on a broken locator; find why a UI test is red before editing it; a fresh context does not isolate server data; log in once with saved state; reload to prove a save; `exact: true`; one-time reads don't retry; route before action; fake failures and assert what the user sees; trace on failure; keyboard and ARIA checks; how to run axe |
| qa-skills-main `skills/bug-reproduction`, `exploratory-testing`, `ai-bug-triage`, `release-readiness` and others in that group | same | State-transition probes and non-numeric boundaries (offered for test-cases, not yet added); bug reproduction methods kept for the debug skill |
| qa-skills-main `skills/payment-testing`, `email-testing`, `chaos-engineering`, `ai-system-testing`, `test-environments`, `testing-in-production` | same | Webhook cases; confirm payment on the server; provider-side clocks; email through a local SMTP catcher; fault injection without tools; LLM feature checks; the same database engine as production; flag on and off |
| addyosmani/agent-skills `skills/test-driven-development`, `references/testing-patterns.md` | MIT, Copyright (c) 2025 Addy Osmani | Red for the right reason; choosing the level; real, then fake, then stub, then mock; a rollback test hits the real thing |
| wshobson/agents `plugins/tdd-workflows`, `unit-testing`, `api-testing-observability`, `python-development/skills/python-testing-patterns`, `javascript-typescript/skills/javascript-testing-patterns`, `developer-essentials/skills/e2e-testing-patterns` | MIT | Retry tests assert the call count in three cases; fake timers start the promise, advance, then await |
| Installed `testing-unit-integration` and `testing-e2e` (the user's, from ~/.agents/skills) | as installed | Every value in an assertion appears in the setup; negative checks; the least-privileged role; a bug fix changes the existing test; typed stubs from real responses; find elements as a user does. This skill replaces both |
| Installed `testing-unit-integration` (SKILL.md, references/fake-timers.md) and `testing-e2e` (SKILL.md), read in full by me on 2026-11-02 | as installed | For `unit.md` and `e2e.md`: the arrange-act-assert shape, a flat test body, no assertions in helpers or hooks, typed inputs with a visible cast, the good and bad example tests (rewritten), whole-value comparison, the error type plus context, two items and an excluded item, both states of a boolean flag, dangerous builder defaults, the doubles table, reset in `beforeEach`, the integration rules (real router, assert through the public API, several records, cascades, `expect.any`), the fake-timer order (start, advance, await) and `rejects`, journey-step helpers, asserting the hand-off to a third party |
| qa-skills-main `skills/unit-testing/SKILL.md` and `skills/playwright-automation/SKILL.md` (lines 1-200), read by me | per that repo's LICENSE | The doubles taxonomy wording, `expect.hasAssertions()`, snapshots only for structured output with `--ci`; page objects only when several tests drive the same page, objects that return values and never assert, composition over a base class, `test.step` for long journeys |
| Author's library rules | n/a | Features: code first, then tests, each proven by breaking the guarded code; bugs: the reproduction test first; a repo with no tests: ask once; the error hunt with edge cases hardest; the edge bank by input type with a proving break per row; coding (formerly no-slop) and test-cases (formerly cases-to-run) as the writing and listing skills |

## What was dropped, and why

- Every coverage target and mutation-score band (80%, 90% on new code, 75-85%, 95%/90% per directory, 50/60/70%), and the test-pyramid ratios (70/20/10, 80/15/5). No source; and coverage targets invite tests written to hit a number.
- Statement and assertion caps ("max 10 statements", "max 3 assertions", "max 15 statements", "arrays default to 2 items"). Invented.
- Priority tags, risk scores, tiers, 13-section strategies, effort tables, KPI tables, and plan documents before code. They conflict with test-cases and with writing documents last.
- Faker data with no seed, factories for every entity, abstract base pages, adapters and API-client wrappers. No helper until the third copy; randomness only with a seed.
- Retries on by default in CI, non-blocking `@flaky` quarantine, longer timeouts as a flake fix, `networkidle`, `force: true`, act-if-visible. They hide failures.
- All testing in production: synthetic orders, production credentials, chaos in production, golden cases from production traffic. The user never touches production.
- Invented numbers: flake-rate bands (<2%/2-5%/>5%), "20" and "50/50" runs, "14 days" quarantine, "30 days" cassette expiry, the 0.85 clustering cutoff, confidence weights, "30-40%" of accessibility issues caught by tools, browser and device share tiers, visual diff thresholds, latency and error-rate targets, kappa 0.6, "8/10" pass rates, "50 cases", "90 minutes" fatigue, and cost multipliers such as "1000x".
- Wrong or doubtful claims found by the readers, among them: old mutmut commands; a single-quoted `$(...)`; `prisma migrate reset` (wipes the database); "Ajv defaults to draft 2020-12"; "route interception is Chromium-only"; `test.info()` inside a worker fixture; a burn rate of 6 on a 1 h window; "LCP, FID, CLS" (INP replaced FID); "OWASP Top 10 (2025)" given as the 2021 list; `run_id` as a metric tag. None is used.
- Vendor material: Healenium, Trunk, PactFlow, Codecov, Percy, Chromatic, BrowserStack, Appium, Detox, Mailosaur, Stripe test card numbers, k6, Checkly, LaunchDarkly, Garak, Promptfoo, DeepEval.
- Bug reproduction (minimise one cut at a time, `git bisect run` with exit 125, "can't reproduce" classes, intake fields, test bug versus app bug). Kept for the debug skill.
- Exploratory charters, the HICCUPS oracles, and release-readiness evidence rules. Offered for test-cases and ship separately; not in this skill.
- Changed from the installed pair: no "max 10 statements", "max 3 assertions", "max 15 statements" or "arrays default to 2 items"; faker only with a fixed seed instead of randomising by default; a build function from the third copy instead of a factory file for every entity; test ids allowed where the repo already uses them instead of banned; `nth()` allowed when order is the behaviour; a one-line provenance comment instead of multi-line JSDoc; route constants and the HTTP-status-to-error-class table dropped (one project's convention); the "When X, then Y" title used only when the repo has no naming style.
- Dropped from qa-skills unit-testing: version news for Jest 30, Vitest 4 and 5, coverage threshold configuration, and the mutation-score bands. From playwright-automation: the abstract `BasePage`, `retries: isCI ? 2 : 0` as a default, and the team-maturity tiers.

## Verified external facts

Checked on 2026-11-02.

| Fact | Where checked |
|---|---|
| `exact`: "Whether to find an exact match: case-sensitive and whole-string. Default to false."; role `name` matching is case-insensitive substring by default | https://playwright.dev/docs/api/class-page |
| `'networkidle'` is marked "DISCOURAGED"; "Don't use this method for testing, rely on web assertions to assess readiness instead." | https://playwright.dev/docs/api/class-page |
| `--repeat-each` "Run each test N times"; `--fail-on-flaky-tests` "Fail if any test is flagged as flaky"; `--trace` accepts `retain-on-failure` | https://playwright.dev/docs/test-cli |
| Log in once in a setup project and reuse `storageState`; add `playwright/.auth` to `.gitignore`; one account per worker for tests that modify server-side state | https://playwright.dev/docs/auth |
| `route.abort` accepts `'timedout'` among its error codes | https://playwright.dev/docs/api/class-route |
| `forbidOnly` "exit with an error if any tests are marked as `test.only`"; `webServer` launches a server during the tests; `retries`; `fullyParallel` | https://playwright.dev/docs/test-configuration |
| `screenshot` accepts `'off'`, `'on'`, `'only-on-failure'`; `trace` accepts `'retain-on-failure'` among others | https://playwright.dev/docs/test-use-options |
| `test.step` "Declares a test step that is shown in the report." | https://playwright.dev/docs/api/class-test |
| axe tags `wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa`, `wcag22aa`; each rule has one version/level tag, and running both 2.0 levels needs both tags | https://github.com/dequelabs/axe-core/blob/develop/doc/API.md |
| `vi.advanceTimersByTimeAsync` exists and includes asynchronously set timers; fake timers wrap `Date` | https://vitest.dev/api/vi.html |

## Not verified, so left out
- The name of MSW's "fail on an unhandled request" option: the fetched page documents `onUnhandledFrame` (default `"warn"`), not the `onUnhandledRequest` the sources name. The skill says to make unstubbed calls fail without naming a flag.
- `context.setOffline`, `route.fulfill({ status })` and `waitForResponse` are used by name from the readers' reports and common Playwright use; not re-fetched.

## Read
qa-skills-main: all 50 skills (184 files), README, AGENTS.md, CLAUDE.md, agents/, docs/WITH-WITHOUT-2026-10-22.md, docs/V3-DESIGN.md. Not read: docs/evidence/, evals/, tests/, tools/, site/, the other docs. agent-skills-main: test-driven-development, browser-testing-with-devtools, references/testing-patterns.md. agents-main: tdd-workflows, unit-testing, api-testing-observability, performance-testing-review, and the e2e, JavaScript and Python testing-patterns skills; not read: bats, temporal, web3 and screen-reader testing.

## Refresh
1. Re-check the Playwright, axe and Vitest pages above before changing `e2e.md` or the fake-timer rule.
2. Look up MSW's current unhandled-request option before naming it.

## Added 2026-11-03
- The "Not for" line points to `prove` for checking a change against an answer key.
