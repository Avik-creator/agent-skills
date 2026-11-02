# Sources

## The author's skill
`one-shape`, pasted in chat on 2026-11-02, is the core. It was first kept word for word. On 2026-11-02 the author asked for it to be rewritten so it doesn't read as a copy. SKILL.md now restates every rule in the library's voice:
- A fuller description, a quick start and an output format.
- Reasons added.
- `no-slop` is now `coding`, and `open-doors` is now `security`.
- No rule was dropped or changed in meaning.

The original text stays below for checking.

The original text, as pasted:

```
---
name: one-shape
description: >-
  Use when asked to refactor, clean up, or standardize a repo. Not for a
  security audit or a single pull request.
---

# One shape

## Scope

The audit is read-only until the user approves a batch. Then apply one step at a time with `no-slop`.

Make the smallest change that removes the drift. Do not add an abstraction a third copy does not force.

Skip code the team does not own: `node_modules`, build output, generated files, vendored code, lockfiles, and migrations. A security issue gets one line pointing to `open-doors`.

## Before auditing

Ask every blocking question in one turn. Never invent a fact. Put an unknown under Open questions and name what it blocks. Replies are 1–2 lines. Write a paragraph only when the user asks.

Ask the target area, hard constraints, and behavior that must not change.

## Standards

1. Read the repo's lint, format, and tsconfig settings, and any convention file such as `AGENTS.md`.
2. Where config is silent, the standard is the pattern most of the repo already uses for naming, file layout, module boundaries, imports, data fetching, validation, and error handling.
3. When the config and the majority pattern disagree, the config wins. When there is no clear majority, add an open question. Do not invent a standard.

## What to find

Each finding cites its files and callers, or the places the logic repeats.

- **Standards drift:** code that breaks the repo's config or its majority pattern.
- **Structure:** a file in the wrong layer, an import that crosses a module boundary, or an inconsistent folder layout.
- **Duplication:** the same logic, matched by behavior, with a copy count. Propose a helper only on the third copy. Two copies count only when they have already drifted apart.
- **Divergent flows:** different paths doing the same job. Unify them on the existing correct version. When copies differ, list the differences and ask which behavior is right.
- **Reimplemented code and over-engineering:** a local copy of the standard library or an installed package, a pass-through wrapper, an interface with one implementation, or an option with one caller.
- **Dead code:** unused exports, files, branches, and dependencies. A search is not proof. Code is still used when a framework entry point (such as Next.js `page.tsx` or a route file), a dynamic import, a string lookup, a config file, a script, or a package export refers to it.
- **Evident performance waste:** a repeated query, work repeated in a loop, or an unbounded load. Report it only when the code shows it.

Never propose a new abstraction, a new dependency, a framework migration, or a split done only for file size.

## Priority

- P0: copies or flows that already disagree and can cause a bug.
- P1: high-traffic drift, duplication, dead code, or waste.
- P2: local polish.

Mark each step's effort and risk as low, medium, or high.

## Plan and apply

Order the steps: delete dead code, then replace reimplemented code, then fix standards drift, then unify flows, then simplify what is left. A formatting-only change is its own step and uses the repo's formatter. Never mix it with a logic change.

Each step is small, keeps behavior the same, ships on its own, names its files, and names the test that checks it when one exists. A repo with no test code does not block a step. A merge keeps every current caller's output the same. If any caller's output would change, that needs its own approved step. On a big repo, the user approves one batch at a time.

Stop after the report. Edit nothing until the user approves a batch.

1. Before the first step, run the repo's lint, typecheck, and tests when the repo has them, and record what already fails.
2. Apply one approved step at a time with `no-slop`. Each step stays its own diff.
3. After each step, run the same checks. If a new failure appears, revert that step, stop, and report.
4. After a batch, re-check the remaining steps, then wait for the next approval.

Do not commit unless the user asks.

## Summary

Counts by priority, the steps applied, the steps left, the areas read and not read, and open questions. Do not claim lines saved or speed gained, and do not call a partial audit complete.

## Replies

Say how many findings, and what needs approval next. After a batch, state what changed and what failed, if anything.
```

## Upstream

| Source | Licence | Taken |
|---|---|---|
| cc-skills-golang `golang-refactoring` (SKILL.md, references safety-net, workflow, catalog, structural, go-tooling) | MIT | Pin as an offered step (safety-net.md:15-16), judging coverage on the touched lines, confirming the test runs them (:13), a move before a speed-up (SKILL.md:73-75), rename with the language tool and report a refusal (:77-79), no text replace for structure (go-tooling.md:123 context), searching strings and config after a rename (SKILL.md:91-93), groundwork steps first (workflow.md:59), the forwarding move (catalog.md:115), rerunning the benchmark (SKILL.md:128) |
| agent-skills `code-simplification` | MIT | Tests pass unedited (:41), read history before deleting (:108-119), what isn't simplifying (:96-97), a single-implementation interface used by a test (:98), the rename or codemod tool (:170), measuring a performance path (:27) |
| wshobson/agents `code-refactoring` / `codebase-cleanup` (byte-identical `refactor-clean.md`, `tech-debt.md`) | MIT | Risk from callers and coverage (refactor-clean.md:557-559, scores dropped), traffic from change frequency (tech-debt.md:46, refactor-clean.md:569), import cycles (tech-debt.md:41), deprecated APIs as drift (:59), commented-out code (refactor-clean.md:900) |
| wshobson/agents `framework-migration` (`legacy-modernize`, `dependency-upgrade`, `code-migrate`, `deps-upgrade`), agent-skills `deprecation-and-migration` | MIT | Parity on the same inputs, callers moved one at a time, the old path unchanged until the new one is complete, removing old tests, config and docs together, codemod dry run first, re-checking the baseline after a revert |

The Cursor sweep reports for 2026-11-02 are in the session scratch folder (`sweep/refactor-*.md`). Seven citations were spot-checked against the sources: six matched, and one was off by two lines (`tech-debt.md:44`, actually `:46`).

## Dropped
- Invented numbers:
  - 1-10 risk scores (refactor-clean.md:557-559)
  - coverage tiers of 40% and 80%, and "two minutes" (golang-refactoring)
  - the 500-line threshold (code-simplification:170)
  - 100-500 lines per PR
  - rollout percentages of 5/25/50/100 and 24-hour waits, and 30 days at zero traffic (legacy-modernize)
  - ROI and dollar figures (tech-debt.md)
- Committing after every green step and a PR per change (golang-refactoring, tdd-refactor). `deps-audit` generating a PR.
- Edits before approval: `npm audit fix --force`, `ncu -u`, Renovate automerge, the codebase-wide sweep in golang-modernize, a regex rewrite across all files.
- `// REFACTOR(step N)` plan comments in code, and a `.modernize` file written into the project.
- Prescriptions for SOLID interfaces, dependency injection, design patterns, gateways and circuit breakers. Framework migrations.
- Extracting on two copies (code-simplification). The library's own `code-review/references/checks.md:58` said the same and was changed to the third copy.
- Broken samples: two code-simplification "after" examples change behaviour (a Map in place of an object, a dropped `async`). The refactor-clean O(n) example doesn't match its "before" and isn't valid Python.
- The Go-only mechanics: 12 items in `sweep/refactor-go-testability.md`, items 15-26. They are held for the go skill until three facts the sources disagree on are checked: generic methods in Go 1.27, `encoding/json/v2` as the default, and `go test` coverage of packages without tests.

## Also changed in this round
- `coding`: new behaviour in uncovered code goes in a new tested function (golang-refactoring catalog.md:189-198).
- `testing`: pull a pure calculation out of I/O only when a test needs it (qa-skills `ai-qa-review/references/testability-refactors.md:21,52`).
- `code-review`: checks.md:58 now uses the third copy, and the description points a repo-wide cleanup to refactor.

## Refresh
1. Re-read the upstream files above.
2. Keep the core in step with the author's own text whenever they revise it.
