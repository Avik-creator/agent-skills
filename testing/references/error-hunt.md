# The error hunt

Read at SKILL.md step 2, and when picking checks in `review`. For each changed function, go down this list and ask "could this be wrong here?". The last column is the break that proves a test catches it (SKILL.md step 5).

## Edge cases, by input type
Hunt these hardest. Take one value per class, and at every limit the last allowed and the first rejected value.

| Input | Edges | Break to prove the test |
|---|---|---|
| Numbers | 0, negative, the limit and one past it, max and overflow, float precision, NaN | move the limit by one |
| Strings | empty, only whitespace, leading or trailing spaces, very long, Unicode and emoji, case, quotes and other injection characters | drop the trim or the length check |
| Collections | empty, one item, duplicates, page size N and N+1, unsorted, very large | remove the empty or dedupe branch |
| Optional data | missing, null, empty versus absent, the wrong type | remove the null guard |
| Time | a DST change, Feb 29, month and year end, time zones, expired versus just valid | compare with `<` instead of `<=` |
| Money and units | rounding, currency, seconds versus milliseconds | round once less, or use the wrong unit |
| Ids and ownership | an unknown id, someone else's id, a deleted record | drop the ownership filter |
| Repeats and order | double submit, a retry, out of order, two at once | remove the idempotency check or the lock |

Expected values follow the server's canonical form: what it trims, lower-cases or sorts, the test expects trimmed, lower-cased or sorted.

## Logic errors
- A wrong or inverted condition; `<` versus `<=`; `&&` versus `||`. Break: flip it.
- Steps in the wrong order (validate after the write). Break: swap them.
- A branch that never runs, or a default that hides a case the code should refuse. Break: delete the branch; the test should notice.
- A calculation checked with one example. Check it with a second composition of the same values (two items versus one item twice).

## Structural errors
- Caller and callee disagree on the contract: units, nullability, an error the caller doesn't expect. Test through the caller.
- An error swallowed or not passed up. Break: replace the error with a silent return.
- A missing call: it saves but never notifies, it charges but never records. Break: drop the call.
- A shared function changed: run the tests of every caller, and add one for a caller whose behaviour now differs.
- A migration: insert rows at the previous version, apply it, and assert the rows survived; run the down migration and re-apply. Use the database skill's migration checks where installed.

## State errors
- A stale read, or a check-then-act gap between two requests. Run two at once.
- State left behind after an error: the next call should see a clean state.
- A save that did not persist: read it back (in a browser, reload) before anything else writes.

## Error handling
- The specific error type and message the caller gets, and that no internals (stack, query) leak.
- A transient error retried, a permanent one not, and giving up after the limit. See failures.md.

## Permissions
- The least-privileged role that should succeed, and the next one down that should fail.
- Another user's or tenant's record: refused, and nothing about it leaks.
- Deeper threat modelling is the security skill's, where installed.
