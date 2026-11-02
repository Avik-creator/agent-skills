# Instrumentation

Read when adding a log line, a metric, or a span. Vendor setup, dashboards, and alert objects belong to that backend's skill.

## Logs

A stable event name plus fields. The message, if the logger wants one, stays a fixed string.

```text
event=payment_failed payment_id=pay_19 provider=gateway error_code=timeout attempt=2
```

Before: `Payment pay_19 failed for user 4 after 2 retries`. After: the line above. The id is a field, so a search does not depend on the sentence.

Pass the correlation id in the header the service already uses. If it has none, generate one at the edge and echo it on the response. A worker that pulls from a queue reads the id from the message. An id invented inside the worker cannot be joined to the call that enqueued the job.

When a scheduler, a replay endpoint, and a manual run write one stream, set `entry_point` where the run starts (`scheduler`, `replay`, `cli`) and copy it with the correlation id. A field you infer later, from the process table or a deploy log, is a hint. Those records are not always still there.

Allowlist fields. Log the dependency name, the status class, the duration, and the attempt. Drop the body, the `Authorization` header, tokens, and direct identifiers (name, email, phone); an opaque user or tenant id may stay. Encode or strip carriage return and line feed in field values; a newline is a forged second line (the security skill).

Spot-check one real line. A field that prints as an opaque object dump is not structured.

## Metrics

RED for an endpoint or a dependency: how many calls, what fraction failed, how long they took. USE for a resource: how full it is, how often work waits, how often it errors.

```text
http_request_duration_seconds{method="POST", route="/payments/:id", status_class="5xx"}
```

`route` is the template. `status_class` is `2xx`, `4xx`, or `5xx`. The same shape for a dependency, with its name as the label.

Do not label with a user id, a tenant id, an email, a request id, a raw URL, or the error text. Those go on the log and the span for that one request.

A histogram has buckets. Read p50, p95, and p99 from it. Do not alert on the mean. Bucket edges are the project's, chosen so the percentile you care about falls inside the range. Do not copy a bucket list from another service and treat it as universal.

## Traces

Use OpenTelemetry and propagate context. On HTTP, that is the W3C `traceparent` header (and `tracestate` when the caller sent it). Checked on 2026-11-02 against the W3C Trace Context recommendation: a participant forwards `traceparent` so the trace is not broken at that hop. Put the same context on a queue message and on work you schedule for later. A span that starts with a new trace id is a different request.

Add a manual span around a unit a person would look up (`charge_provider`, `apply_discounts`), with the attributes on-call will filter on. Do not span every function. Do not put direct identifiers, tokens, or raw URLs in attributes.

Sampling, from the OpenTelemetry sampling docs, checked on 2026-11-02. Pick one and name it. Do not invent a rate.

- **Head-based** decides when the trace starts, from the trace id and a fraction you choose. It is cheap. It cannot prefer traces that later fail or run long, because that is not known yet.
- **Tail-based** decides after the spans have arrived, so it can keep errors or slow traces. It has to hold the spans until it decides.

A high-volume service can do both: head-based first, tail-based later. Say which, and why, in the design doc or the PR. The OpenTelemetry page uses "5%" as an illustration of a fraction. That number is not a default for this skill.
