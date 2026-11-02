# Simplifying a change

Read in two cases: the reviewer reads the signals below while checking readability, and you read the whole file before a simplification pass on your own change.

## Signals (judgement calls only)
Report hits under Judgement calls as "possible <signal>", like the smells in checks.md. The repo's conventions win.
- **Deep nesting**: the main path is buried in conditions. Use guard clauses that return early.
- **Nested ternaries**: the reader has to keep a stack in their head. Use if/else, a switch or a lookup table.
- **Boolean flag parameters**: `send(msg, true, false)` says nothing at the call site. Use separate functions or a named option.
- **The same condition in several places**: give it one well-named predicate.
- **A wrapper that adds nothing**: call the real target.
- **A pattern with one case**: a factory for one type, a strategy with one strategy. Use the direct code.
- **A misleading name**: a `get` that also writes. Rename it to what it does.
- **A function doing several jobs**: split it at the points where you'd want to name a step.

Don't call these "simpler": inlining a helper whose name explained a concept, merging two unrelated functions, or fewer lines that take longer to read. Why: the goal is how fast a new reader understands the code, not its line count.

## The pass (your own change, after a yes)
Offer it after the report when the signals found something. Touch only lines the change added or modified. Why: drive-by refactors widen the diff and the risk.

1. **Understand it first.** What is it responsible for? What calls it, and what does it call? Its edge cases and error paths? Which tests pin its behaviour? Why might it be written this way (`git log -L` or blame)? If you can't answer, read more or leave it.
2. **Behaviour stays identical**: the same outputs, errors, side effects and their order. If you're unsure a change preserves this, don't make it.
3. **One change at a time.** Run the tests after each one, without modifying them. Why: a test you had to change means behaviour changed, and one change at a time tells you which edit broke it.
4. **Compare before and after.** If the result isn't clearly easier to read, revert it. Not every attempt succeeds.
5. **Its own commit**, separate from the feature or fix. Why: a refactor mixed into a behaviour change is hard to review and to revert alone.
6. **Comments follow the repo's rules.** Where there are none, one line that says what the symbol is or does.

Then run the same checks as before the pass, and say in the hand-over which simplifications you made and which you reverted.
