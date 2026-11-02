# Hypotheses and evidence

Read at SKILL.md step 5. The aim is to disprove guesses quickly, not to collect support for the first one.

## Write each hypothesis like this
```
H1: <one sentence: what is wrong, where>
Category: <from the list below>
Evidence for: <file:line, log line, or observation>
Would disprove it: <the observation that rules it out>
Check: <the one probe that tells them apart>
Result: confirmed | disproved | inconclusive
```
Spread the first two or three across different categories. Why: three guesses from the same category share one blind spot.

## Categories
- **Logic**: a wrong operator or condition, off by one, a missing case, a wrong order of steps.
- **Data**: an unexpected shape, null or empty, a wrong type or coercion, encoding, truncation, overflow, a duplicate.
- **State**: a race, a stale cache, wrong initial values, shared state changed by someone else, an illegal state transition.
- **Integration**: a request or response that doesn't match the contract, version skew between parts, a config that differs between environments, a missing variable.
- **Resource**: a pool, file handle, memory or disk running out, a lock held too long, a timeout.
- **Environment**: a missing or different runtime or dependency version, permissions, platform behaviour, time zone or locale.

## Grading evidence
| Kind | Weight | Example |
|---|---|---|
| Direct | strong | `orders.ts:42` checks `> 0` where the rule says `>= 0`, and the repro hits that line |
| Correlation | medium | errors began after commit `abc123` was deployed |
| Absence | depends on the search | no null check on this path; say what was searched |
| Someone's report | weak | "it works on my machine" |

- Record the evidence against a hypothesis as well as the evidence for it.
- A confirmed cause has a chain from the cause to the symptom, with every link observed or cited.
- When two hypotheses are both confirmed, check whether they are one compound cause before ranking them.
- When none is confirmed, write new ones from what the checks showed. Don't widen the old ones.

## Reading a stack trace
- Start at the throw, then find the first frame in this repo's code. That is usually where to look, even when the throw is inside a library.
- A null or missing value inside a library usually came from this code's arguments.
- A timeout in an HTTP or database client means the other side was slow or the wait was too short. Find out which before calling it a cause.
- A "modified during iteration" error means the collection changed during the loop, often from another thread or a callback.
- Async stacks lose frames. Look for the code that started the operation, not only the frame that caught it.

## Compare with what works
Find the nearest working case: a similar function, the same flow with another input, the same request in another environment, the last good build. Lay the two side by side and list every difference: code, config, versions, data, user, time. Test the differences one at a time.

## Causes outside the code
Check these before blaming the code that changed:
- The API contract changed: a new required field, a different response shape, a removed endpoint.
- Schema drift: a migration not applied, a changed type or constraint.
- An expired or rotated credential, token or certificate.
- DNS, a firewall or network policy, a proxy or load balancer sending traffic to the wrong place.
- Rate limits or quota.
- A degraded upstream service, or one returning stale cached data.
- A queue that is full or lagging.
- Clock skew affecting token expiry, cache lifetimes or schedules.
- A full disk, a read-only filesystem, changed file permissions.
- A process killed for memory, or a wrong image tag or missing environment variable in the deploy.
- A dependency upgraded with a change in behaviour.

## Logs
- Before calling two log lines the same failure, strip numbers, ids, URLs, paths and timestamps. Then compare the message and the top frames.
- A log line is evidence of what the code wrote, not of what happened. Confirm it against the data or the response.
