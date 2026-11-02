---
name: refactor
description: >-
  Audits a repo or an area of it for drift, duplication, divergent flows,
  reimplemented code, over-engineering, dead code and visible waste, then
  applies approved fixes one behaviour-preserving step at a time. Use when
  asked to refactor, clean up, standardize, deduplicate, remove dead code, or
  make an area consistent. Not for a security audit - use security; not for
  reviewing one pull request - use `review`; not for a new feature or a
  bug fix - use coding.
---

# Refactor

Find where the code has drifted from its own standard, propose small steps that bring it back, and change nothing until the user approves a batch. Every step keeps behaviour exactly as it is.

## Quick start

1. Ask for the target area, the constraints, and the behaviour that must not change.
2. Work out the repo's standard from its config, then from its majority pattern.
3. Audit read-only, and rate each finding P0, P1 or P2, with effort and risk.
4. Report, then stop.
5. After a yes, apply one step of the approved batch at a time with `coding`, checking after each.

## Scope

- **Out of scope:** code the team doesn't own: `node_modules`, build output, generated files, vendored code, lockfiles, and migrations. A security problem gets one line pointing to `security`, not a fix.
- **Smallest change:** take the change that removes the drift. A new abstraction needs a third copy that forces it.
- **Never propose:** a new abstraction, a new dependency, a framework migration, or a split done only for file size. Why: each one is a design change dressed as cleanup.

## Before the audit

Ask every blocking question in one turn. Never invent a fact. Put an unknown under Open questions and say what it blocks. Replies are 1–2 lines; a paragraph only when the user asks. Ask for the target area, the hard constraints, and the behaviour that must not change.

## Workflow

### 1. Find the standard
- Read the lint, format, and TypeScript or compiler settings, and convention files such as `AGENTS.md`.
- Where config is silent, the standard is what most of the repo already does: naming, file layout, module boundaries, imports, data fetching, validation, error handling.
- Config beats the majority. With no clear majority, raise an open question. Why: an invented standard is a new rule nobody agreed to.

### 2. Audit
Every finding cites its files and callers, or each place the logic repeats. What counts for each kind is in [findings.md](references/findings.md).
- **Standards drift:** code that breaks the config or the majority pattern.
- **Structure:** a file in the wrong layer, an import across a module boundary, or folders laid out inconsistently.
- **Duplication:** the same behaviour, with the copy count. A helper only on the third copy; two copies count only once they have drifted apart.
- **Divergent flows:** different paths doing the same job. Unify on the version that is already correct. When the copies differ, list the differences and ask which behaviour is right.
- **Reimplemented code and over-engineering:** a local copy of the standard library or an installed package, a pass-through wrapper, an interface with one implementation, or an option with one caller.
- **Dead code:** unused exports, files, branches and dependencies. A search is not proof. Code is still live when a framework entry point (a route or page file), a dynamic import, a string lookup, a config file, a script, or a package export reaches it.
- **Visible waste:** a repeated query, work redone in a loop, or an unbounded load. Report it only when the code shows it.

### 3. Rate
- **P0:** copies or flows that already disagree and can cause a bug.
- **P1:** drift, duplication, dead code, or waste in high-traffic code.
- **P2:** local polish.
- Each step also gets effort and risk: low, medium or high.

### 4. Plan
- **Order:** delete dead code, replace reimplemented code, fix drift, unify flows, then simplify what is left. Why: each stage shrinks what the next one has to read.
- **Formatting:** a formatting-only change is its own step, done with the repo's formatter, and never mixed with logic.
- **Each step:** small, keeps behaviour, ships on its own, and names its files and the test that checks it. A repo with no tests doesn't block a step.
- **Merges:** a merge keeps every current caller's output the same. A change to any caller's output is its own approved step.
- **Batches:** on a big repo, the user approves one batch at a time.
- How each step stays safe: [safe-steps.md](references/safe-steps.md).

### 5. Report and stop, editing nothing until the user approves a batch

### 6. Apply an approved batch
1. Run lint, typecheck and tests where the repo has them, and record what already fails.
2. Apply one step with `coding`, as its own diff.
3. Run the same checks. On a new failure, revert the step, stop, and report.
4. After the batch, recheck the remaining steps against the changed code, then wait for the next approval.

Commit only when the user asks.

## Output format

```
Findings: P0 <n>, P1 <n>, P2 <n>
Read: <areas>     Not read: <areas>
Steps
- [P<n>] <what> - <files> - effort <l/m/h>, risk <l/m/h> - check: <test or command>
Applied: <steps> (after a batch)     Left: <steps>
Open questions
- <question> - blocks <step>
```

Never claim lines saved or speed gained, and never call a partial audit complete.

## Replies

Say how many findings and what needs approval next. After a batch, say what changed and what failed.

## References

- [references/findings.md](references/findings.md) - what counts as each kind of finding, and how to rate risk and traffic.
- [references/safe-steps.md](references/safe-steps.md) - read before applying a step: pinning behaviour, moving callers, renames, ordering, and reverting.
