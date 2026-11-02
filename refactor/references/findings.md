# What counts as a finding

Read while auditing (SKILL.md "What to find"). Each item sharpens one kind of finding. A finding still cites its files and callers.

## Rating risk and traffic
- **Risk** comes from two things you can check: how many callers the code has, and whether a test runs the lines the step changes. Judge coverage on those lines, not on the package or repo average. Many callers and no test is high; few callers and a test that runs the lines is low.
- **High-traffic** (P1) means the file changes often in git history (`git log --format=%h -- <file>`), or it has many callers. Say which one applies.
- Give the basis with the rating, for example "risk high: 14 callers, no test reaches the branch".

## Structure
- An import cycle between modules is a structure finding. Its fix is usually a step that others depend on (safe-steps.md "Order").

## Dead code
- Commented-out code is dead code.
- Before calling code dead or over-built, read why it exists: `git log -L` on the function, or blame. Put the reason in the finding when it changes the call, for example a workaround for a platform bug that still exists.

## Standards drift
- A call to an API that the installed version marks deprecated is drift. Fix it with the replacement that same version documents. Never fix it by upgrading, since an upgrade is a separate change the user asks for.

## Over-engineering
- An interface with one production implementation is not over-engineering when a test swaps in a stand-in. Check the tests before reporting it.
- These are not simplifications: inlining a helper whose name explains a concept, or merging two functions that only look alike. The review signals in the `review` skill's `simplify.md` apply here too.

## Performance waste
- Report it only when the code shows it. Simplifying a path where speed matters needs a measurement showing it isn't slower; without one, leave the path alone.
