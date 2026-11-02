---
name: observability
description: Instruments a feature so on-call can tell that it broke, where time went, and why, using logs, metrics, and traces, and designs symptom alerts. Includes a measured performance loop and caching rules. Use when adding logging, logs, structured logs, metrics, tracing, OpenTelemetry, spans, a correlation or request ID, alerts, an SLO, or on-call, when asking how we will know it broke, when instrumenting a feature, endpoint, job, or integration, or when the user mentions performance, a slow endpoint, latency, profiling, optimize, or caching. Not for diagnosing a live incident - use the debug skill where installed; not for creating alerts or dashboards in a specific backend - use that backend's skill; not for database query tuning - use database.
---

# Observability

Write the telemetry with the feature. A signal that answers no on-call question is noise. Creating or changing an alert, a dashboard, or a saved view in a real backend is that backend's skill, and only after a yes. Why: shared monitoring is outward-facing.

## Quick start

1. Write 2–4 questions on-call will ask. Map every signal to one of them.
2. Pick the signal: metrics say that something is wrong, traces say where, logs say why.
3. Log, count, and trace as below. Detail is in [instrumentation.md](references/instrumentation.md).
4. For a new alert, follow [alerts.md](references/alerts.md); for an SLO behind it, [slo.md](references/slo.md).
5. In staging, or a local run with the same telemetry, cause a failure and find it from the telemetry alone.
6. A measured slowness follows [performance.md](references/performance.md). Do not start one from a guess.

## On-call questions

```
Feature: checkout payment retry
1. What fraction succeed on the first attempt, and what fraction after a retry?
2. When one fails for good, why (provider error, timeout, validation)?
3. Is the provider slower than usual?
```

If you cannot name the questions, do not add telemetry yet.

## Logs

Log an event with a stable name and structured fields, not a sentence with the values interpolated in. Levels:

| Level | When | On-call |
|---|---|---|
| error | An invariant broke; someone may have to act | Investigate |
| warn | Degraded, and the code handled it | Watch the trend |
| info | A business event (order placed, job finished) | None |
| debug | Diagnostic detail | Off in production |

Generate or accept a correlation id at the boundary and pass it on every log line, outbound call, and queue message. When more than one entry point writes the same log, stamp which one started the run, next to the correlation id. Allowlist the fields. No secrets, tokens, bodies, auth headers, or direct identifiers (name, email, phone); an opaque user or tenant id is a field, as the security skill's privacy rules say.

The security skill's audit events are info (or warn on failure): login success and failure, role and permission changes, data export, data delete.

## Metrics and traces

RED (rate, errors, duration) on each endpoint and each dependency. USE (utilization, saturation, errors) on a resource such as a queue, a pool, or a host. Labels come from a bounded set: route template, status class (`5xx`, not `503`), dependency name. A user id, tenant id, request id, URL, or error text is a log field, not a label. Why: each new label value is a new series.

Record latency as a histogram and read it as a percentile. An average hides the slow tail.

Trace with OpenTelemetry. Propagate W3C `traceparent` across HTTP, queues, and async work. A manual span wraps one meaningful unit of work. No direct identifiers or tokens in span attributes. Sampling is a choice of head-based or tail-based, described in [instrumentation.md](references/instrumentation.md), not a default rate.

SDK setup and exporters follow the language's OpenTelemetry docs; for Go logging (`log/slog`) see the go skill.

## Verify

In staging, force the failure the questions are about. Find that one request by its correlation id, see the metric move, and see the trace cross the hop. Do this before calling the feature done. Why: telemetry is code, and it can be wrong.

## Hand over

- The on-call questions, and which signal answers each.
- What you induced, and where it showed up.
- **Could improve:** a missing RED series, a label with unbounded values, an alert with no runbook. Propose it, and apply it after a yes. A change inside a monitoring backend waits for that backend's skill and a yes.

## References

| File | Read when |
|---|---|
| [instrumentation.md](references/instrumentation.md) | adding a log line, a metric, or a span |
| [alerts.md](references/alerts.md) | writing or changing an alert |
| [slo.md](references/slo.md) | defining an SLI, SLO or error budget, or a burn-rate alert |
| [performance.md](references/performance.md) | a measured or reported slowness, or a stated target |
