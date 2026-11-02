# Testing failure paths

Read when the change retries, deduplicates, takes webhooks, has timeouts, sends email, or calls a model. All of these run locally or against test setups, never production.

## Retries
Assert the call count in three cases: a transient error is retried, the limit gives up with the right error, and a permanent error is not retried at all.

## Duplicates and idempotency
- Send the same request twice at once: one durable effect.
- The same idempotency key with a different payload: refused, not silently accepted.

## Webhooks
- Missing, wrong or tampered signature: refused, and nothing is recorded. Compute the signature over the raw body bytes, and test a body with the same JSON but different bytes.
- The same event delivered twice, or twice at once: handled once.
- An older event arriving after a newer one cannot undo a final state.
- A crash after recording "seen" but before the work: the event can still be processed.
- A payment is confirmed by the verified event on the server, not by the browser redirect.

## Timeouts
Make the dependency slow, and assert both the timeout behaviour and that the stub was actually reached. Why: a timeout test that never reaches the stub passes for the wrong reason.

## Injected faults, without extra tools
- Point the dependency at a port you have shown is closed.
- Restart the process between two steps of a flow, on the same data, and check the session and the idempotency key survive.
- A rollback or crash test hits the real thing: a real transaction against a real database, a real process exit. An injected exception is not a crash.

## Time and randomness
Freeze the clock and seed the random source. A billing or provider clock that runs on the provider's servers can't be moved by a fake clock; use the provider's test-mode clock.

## Email
Capture the real message with a local SMTP catcher. Use a unique recipient per test, poll rather than sleep, and read only messages for that recipient. Extract exactly one code or link and check its origin. Prove the test by breaking the template.

## LLM features
- Assert properties or a schema, never exact wording.
- Run at the temperature production uses. Where output varies, run N times; the pass rate is the user's decision.
- An LLM used as a judge is checked against human labels before you trust it.
- Check tool choice: the right tool, no tool call for small talk, and the tool's error and timeout paths. Treating model output as untrusted is the security skill's.
