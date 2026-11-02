# Keeping each step safe

Read before applying an approved step. Each step keeps behavior the same: the same outputs, errors, side effects, and their order.

## Pin behavior first (offered, not required)
- When a step changes lines that no test runs, offer a separate step that pins what those lines do today, bugs included. The pin records behavior; it does not judge it. The user may decline, and the step goes ahead without it. Why: a repo with no tests doesn't block a step.
- The pin is written with `testing`. It passes against the current code before the refactor starts.
- Confirm the named test actually runs the lines the step changes: a coverage run on them, or a deliberate break that turns it red. A test that only imports the module gives false comfort.
- Code that looks untested may be dead. Check whether it is reachable before pinning it; delete it instead if it is not.

## A test that had to change
Existing tests pass without edits. If a step only passes after a test is edited, the step changed behavior: revert it and report.

## Two paths for the same job
- Prove parity: run the same inputs through the old and the new path and compare the outputs, including errors.
- Leave the old path unchanged until the new one handles every case it handles.
- Move callers one at a time, and run the checks after each.
- Remove the old path in the same step that removes its tests, config and docs.

## Moving a symbol with many callers
In one approved batch:
1. Add the new home.
2. Keep the old name forwarding to it. The forwarder is temporary, so it isn't a pass-through-wrapper finding.
3. Move the callers.
4. Delete the old name.

## Renames and mechanical changes
- Use the language's own rename or refactor tool (an IDE or language-server rename, the compiler's fix tool). When it refuses, treat that as a real conflict and report it; don't work around it by hand.
- Never apply a structural change with `sed` or a text replace. Why: it also rewrites matches inside strings, comments and unrelated names.
- After a rename, also search string lookups, serialized field names, templates and config, which the tool can't see. Then search for the old name again: zero hits, or each one explained.
- A codemod the maintainers publish: run its dry-run or preview mode first, show the diff, and run it only after a yes.

## Order
- A step that other steps depend on, such as breaking an import cycle or adding a forwarder, runs first, even before the category order in SKILL.md.
- A move and a speed-up to the same code are two steps, the move first. The move is checked by the build and tests, the speed-up by a measurement.
- When the path has a benchmark, rerun it after the step, and stop on a slowdown beyond its normal run-to-run variation.

## After a revert
Run the checks again and confirm they match the baseline recorded before the first step. A revert that leaves a new failure gets reported as that.
