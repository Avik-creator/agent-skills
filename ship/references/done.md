# The done gate

Read at SKILL.md step 2. Every item gets an answer: done (with the evidence), not applicable (with the reason), or missing. Why: an item skipped in silence looks the same as one that was checked.

## Gate
- **It ran.** The changed behaviour was run, not only compiled or type-checked. Give the command and what it showed. A pass that came only on a retry, or a result left by an earlier run, is not a pass. Report anything failed, skipped, blocked or not run as that, never as done. A UI change is checked in a real browser with the automation skill, where installed.
- **Tests.** The new behaviour has a test that fails without the change; a proof test from `review` counts. The full suite passes. Give the command.
- **Checks.** The project's own lint, type and format commands pass. For Go, the go skill's checks.
- **Scope.** Every changed line traces to the request. No unrelated refactor, no leftover debug output, commented-out code or unused imports. After a rename or a signature change, search for the old name again: zero hits, or each one explained. Re-read the whole diff once before the gate.
- **Integration.** Migrations, config, environment variables and flags the change needs are in the change or listed in the description. A changed API or schema states what happens to existing callers or data.
- **Docs.** Visible behaviour that changed is reflected in the README, API docs or help text.
- **Security and telemetry.** Where the change crosses a trust boundary, the security skill's checks were done; where it adds a path on-call must see, the observability skill's. Both where installed.
- **Comments** follow the repo's own rules.

## bar-guard
`scripts/bar-guard [base]` lists moves in the change that lower the bar. The default base is the merge-base with the default branch, and it includes uncommitted and untracked files. Exit 0 is clean, 1 means hits, 2 means it could not run. Treat a 2 as unchecked, never as clean.

What it reports:
- **Added suppressions:** `@ts-ignore`, `@ts-expect-error`, `eslint-disable`, `biome-ignore`, `# noqa`, `# type: ignore`, `//nolint`, `nosemgrep`, `gitleaks:allow`, `istanbul ignore`, `c8 ignore`, `#[allow(`.
- **Added skips:** `.skip(`, `.only(`, `xit(`, `xdescribe(`, `pytest.mark.skip`, `t.Skip`, `#[ignore]`.
- **Added stubs:** "not implemented", `unimplemented!`, `todo!(`, `panic("TODO`, a `TODO`, an empty `catch {}`.
- **Tests weakened:** a deleted test file, or an assertion line removed from a test file that still exists.

Each hit needs one of these:
- **Fix it**: remove the suppression, make the test pass, finish the stub.
- **Keep it with a reason the user agrees to**: put the reason in the code next to it, in the repo's comment style, and in the PR or MR description.
- **It isn't a lowered bar**: for example, the assertion moved to another test. Say where it went.

A threshold lowered in a lint, coverage or CI config is not in the script. Check those files by eye when the diff touches them. Why: their formats differ too much for one pattern.

Extend the patterns in the script for a language it misses; don't write a second checker.
