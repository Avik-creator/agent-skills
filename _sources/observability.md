# Sources

Ideas were adapted and re-checked, not copied. Pulled 2026-11-02.

| Upstream | License | Taken |
|---|---|---|
| addyosmani/agent-skills `skills/observability-and-instrumentation/SKILL.md` and `references/observability-checklist.md` | MIT, Copyright (c) 2025 Addy Osmani | On-call questions first; metrics say that, traces say where, logs say why; stable event names and structured fields; the four levels and what on-call does; correlation id at the boundary; an entry-point field when several starters share a log; allowlisted fields; RED and USE; bounded labels and status class; histograms read as percentiles; OpenTelemetry with context across HTTP, queues, and async work; manual spans around meaningful units; symptom alerts; two severities; threshold from an SLO or history; a runbook link; test-fire; prove it by inducing a failure |
| addyosmani/agent-skills `skills/performance-optimization/SKILL.md`, `references/optimization-patterns.md`, and `references/performance-checklist.md` | MIT, same | Start from a measurement or a stated target; the trimmed "what is slow" tree; baseline, one change, re-measure the same way, compare to run-to-run spread; the keep-or-revert table; an attempt ledger that includes reverts; a guard on the metric that justified the fix; cache only what is expensive and read much more than it changes; key every input the response varies on; one staleness story; one invalidation strategy; stampede as serve-stale or coalesce; never cache balances, permissions, or inventory at checkout |
| wshobson/agents `plugins/observability-monitoring` (all 15 files, read by a subagent; SLO material re-checked against the SRE Workbook by me) | MIT | Only the ideas: SLI as a ratio, error budget = 100% − SLO, burn rate, multiwindow multi-burn-rate alerts, low-traffic handling, an error budget policy as an agreed decision. Every number in `references/slo.md` comes from the Workbook, not this plugin |
| Author's library rules | n/a | Vendor-neutral instrumentation; real backend objects handed to that backend's skill after a yes; runbook shape points at design-docs rather than a copy; Go logging points at the go skill; the debug skill is named for a live incident |

## What was dropped, and why

- "Tracing cost is trivial" and "I'll add logging after" rationalizations that assert a cost without a measurement. Not used.
- Head-based "at a low rate" and "keep 100% of errors" as defaults. Sampling is named as head-based or tail-based. The OpenTelemetry page's "such as 5%" is quoted in the reference as an illustration on that page, not as this skill's rate.
- The three-line runbook template and `docs/runbooks/`. design-docs `references/documentation.md` already defines a runbook (symptom, checks, fix, confirm, who to call). This skill points there.
- Dashboard checklists and a pre-launch gate that assumes a specific aggregator. Creating those objects is the backend's skill.
- Vendor clients and auto-instrumentation snippets (a Prometheus client, a Node SDK bootstrap). The rules are the same without them.
- Performance budgets from the upstream skill: 200 KB JavaScript, 50 KB CSS, 200 KB images, 100 KB fonts, 200 ms p95, 3.5 s time to interactive, Lighthouse ≥ 90. Invented there. Not used. The only numbers are Google's published Core Web Vitals thresholds, in the table below.
- "Users won't notice 100 ms" and any conversion-rate claim. Not used.
- N+1, index, and pool tactics. Those are the database skill. Go `benchstat` is the go skill's `references/testing.md`.
- The long-task "50 ms" cutoff and bundle-size targets. The tree says where to look (TTFB, long tasks, the bundle) and does not set a number.
- From observability-monitoring: Prometheus, Alertmanager, Grafana, Jaeger, Tempo and Fluentd config and PromQL (vendor-specific); service tiers with fixed targets, progressive target phases, template targets like 500 ms/95% (invented); the toil formula (invented); its `burn_rate` = consumed/budget and "days until exhausted" formulas, `avg_over_time` of a ratio (mathematically wrong); "3× over 6 h = 10%" (it is 2.5%), swapped short/long windows, a 1-day/3× tier not in the Workbook; the 6× tier as a warning (the Workbook pages on it); "sample 1-10%", "<1% CPU overhead" (invented); `user_id` span tags (breaks the identifier rule); fixed CPU/memory/disk thresholds (cause alerts); LCP/FID/CLS (FID was replaced by INP); the four agent personas and SLO governance ceremony.
- RED and USE are used as names with their letters expanded. The original essays were not re-fetched, so they are not cited as external facts.

## Verified external facts

Checked on 2026-11-02.

| Fact | Where checked |
|---|---|
| Core Web Vitals good targets: LCP within 2.5 s, INP 200 ms or less, CLS 0.1 or less, judged at the 75th percentile of page loads, mobile and desktop separate. LCP, INP, and CLS are the stable set on that page (page updated 2024-10-31) | https://web.dev/articles/vitals |
| LCP: good is 2.5 s or less, poor is greater than 4.0 s, in between needs improvement (page updated 2025-09-04) | https://web.dev/articles/lcp |
| INP: ≤ 200 ms good, above 200 ms and ≤ 500 ms needs improvement, above 500 ms poor | https://web.dev/articles/inp |
| CLS: 0.1 or less good, poor greater than 0.25, in between needs improvement | https://web.dev/articles/cls |
| W3C Trace Context: `traceparent` identifies the incoming request; a participant forwards `traceparent` and `tracestate` so the trace is not broken | https://www.w3.org/TR/trace-context/#traceparent-header |
| OpenTelemetry sampling: head-based decides at the start and cannot see a later error or the full latency; tail-based decides from the spans. The "5%" sentence on that page is an example of a fraction, and the skill says so | https://opentelemetry.io/docs/concepts/sampling/ |
| SLI is "the number of good events divided by the total number of events"; SLI types table (availability, latency, quality; freshness, correctness, coverage; durability); "five or fewer"; error budget is "100% minus the SLO"; measure at the load balancer or client; the error budget policy is agreed by product, development and SRE, and names who acts and an escalation path | https://sre.google/workbook/implementing-slos/ |
| "Burn rate is how fast, relative to the SLO, the service consumes the error budget"; the 14.4/1h/5m, 6/6h/30m (page) and 1/3d/6h (ticket) table for 99.9% over 30 days; the short window is 1/12 of the long one; budget spent = burn rate × window ÷ period; the three low-traffic options; a 90% target where a full outage spends 1.4% in an hour; 99.999% exhausted in 26 seconds | https://sre.google/workbook/alerting-on-slos/ |

The health-check exclusion in `slo.md` is the library's own rule; the Workbook does not mention it.

## Not verified, so left out

- Any sampling percentage, histogram bucket list, or latency budget as a universal default.
- A Go OpenTelemetry SDK snippet. The go skill documents `log/slog` and does not document that SDK, so the skill points at the go skill for logging only.
- Vendor query languages and dashboard schemas.


## Refresh

1. Re-open the four web.dev pages and update the Core Web Vitals table and the "Checked on" line if a threshold moved.
2. Re-read the W3C Trace Context recommendation if the header name or the forwarding rule changes.
3. Re-read the OpenTelemetry sampling page before adding any rate. Do not promote the page's example percentage into a default.
4. Re-read both SRE Workbook chapters before changing the SLI types or the burn-rate table in `slo.md`.
