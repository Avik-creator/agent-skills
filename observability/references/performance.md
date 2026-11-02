# Performance

Read when there is a measured problem, a reported one, or a stated target. Do not optimize from a suspicion. Indexes, query plans, pools, and migrations are the database skill. Go benchmarks and `benchstat` are the go skill's `references/testing.md`.

## Where to measure first

```
What is slow?
├── Every endpoint → the pool, CPU, memory
├── One endpoint → its queries and its outbound calls
├── Intermittent → locks, GC, a dependency
└── The page → TTFB, long tasks, the bundle
```

Measure that thing. A profiler you happened to have open is not the measurement.

## The loop

1. Record a baseline: the command, the conditions, the number.
2. Find the one bottleneck that baseline shows.
3. Change that one thing.
4. Measure again with the same command and the same conditions. A cold cache against a warm one measures the cache.
5. Repeat the run and compare the delta to the run-to-run spread. A gain inside that spread is not a gain.

| Result | Action |
|---|---|
| Within the noise | Revert |
| Worse | Revert |
| Faster, and a test broke | Revert |
| Faster, tests green, outside the noise | Keep |

Put the before and after numbers in the commit or the PR. An optimization that wins by skipping a check the product needs is a revert even when the number moved.

## Attempt ledger

Write every attempt, including the reverts, in the PR description, or in the file the repo already uses for this. Do not create a new performance log unless asked.

| Idea | Baseline → result | Verdict | Why |
|---|---|---|---|
| Memoize the row | 240 ms → 235 ms | reverted | Inside the ±15 ms spread |
| Virtualize the list | 240 ms → 90 ms | kept | The long task is gone |

The next change reads the ledger before proposing an idea that already failed. The numbers in the table are an example of the shape, not a target.

## Guard

Add a benchmark or a monitor on the metric that justified the fix, the same one, not a nearby one. When it fires, start again at the baseline. Do not add a budget the project did not set.

## Caching

Cache a result that is expensive and is read far more often than it changes. The key includes every input the response varies on: tenant, user, permissions, locale, flags. State how stale a hit may be, and one way it becomes fresh (a TTL, an event, or a version on the key). On a stampede, serve the stale value or collapse the concurrent misses into one recompute. Do not cache a balance, a permission, or inventory at the moment of checkout. Why: a key that omits the viewer serves one person's data to another, and a cached permission is a stale authorization.

## Core Web Vitals

Google's published thresholds, checked on 2026-11-02. "Good" is the 75th percentile of page loads, looked at separately for mobile and desktop. These are field targets for a page, not a budget for an API.

| Metric | Good | Needs improvement | Poor |
|---|---|---|---|
| LCP | ≤ 2.5 s | > 2.5 s and ≤ 4.0 s | > 4.0 s |
| INP | ≤ 200 ms | > 200 ms and ≤ 500 ms | > 500 ms |
| CLS | ≤ 0.1 | > 0.1 and ≤ 0.25 | > 0.25 |

Sources: `web.dev/articles/vitals` (the 75th percentile and the good targets; page updated 2024-10-31), `web.dev/articles/lcp` (updated 2025-09-04), `web.dev/articles/inp`, `web.dev/articles/cls`. Re-check the pages before treating a number as current. A lab score is not a substitute for the field percentile.
