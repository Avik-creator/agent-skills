# Transactions and concurrency

Read when a change reads then writes, updates counters or balances, processes a queue, retries, or touches more than one row that must stay consistent.

## Contents
- Read-then-write races
- Locking and queues
- Isolation levels
- Retries
- Side effects and the outbox
- Idempotency
- Polling a table for new rows

## Read-then-write races

Under READ COMMITTED (the default), `SELECT` a value then `UPDATE` it based on what you read, and a concurrent writer's change gets silently overwritten.

```sql
-- wrong: read balance in app, write balance - 200
-- right: let the row do the arithmetic
UPDATE account SET balance = balance - $3
WHERE tenant_id = $1 AND id = $2 AND balance >= $3;
```

- Check rows affected. `UPDATE 0` means the guard failed or the row is gone. Treat it as an outcome, not as success.
- Reserving stock, seats or quota works the same way: one conditional `UPDATE ... WHERE available >= $n`, never "check availability, then reserve".
- Uniqueness ("does this email exist?") belongs to a `UNIQUE` constraint plus handling of 23505, not to a prior SELECT.
- An `UPDATE` that waits on a locked row re-checks its `WHERE` after the lock is released. It may then skip the row, so again, check the count.

## Locking and queues

Lock rows you must read and then change in the same transaction:

```sql
SELECT balance FROM account WHERE tenant_id = $1 AND id = $2 FOR UPDATE;
```

Work queue with many workers:

```sql
WITH next AS (
  SELECT id FROM job
  WHERE status = 'pending'
  ORDER BY created_at
  LIMIT 10
  FOR UPDATE SKIP LOCKED
)
UPDATE job SET status = 'running', started_at = now()
WHERE id IN (SELECT id FROM next)
RETURNING id;
```

- Lock rows in one consistent order (by id) across all code paths. Inconsistent order is the usual deadlock cause.
- Keep transactions short. No HTTP calls, file I/O or waiting on users inside one. Long transactions hold locks and stop vacuum from cleaning dead rows anywhere in the database.
- Session advisory locks (`pg_advisory_lock`) break behind a transaction pooler. Use `pg_advisory_xact_lock`.

## Isolation levels

- Default to READ COMMITTED plus atomic statements or `FOR UPDATE`.
- REPEATABLE READ gives one snapshot for multi-query reports, and errors (instead of silently overwriting) on a concurrent update.
- SERIALIZABLE is for invariants spanning rows (write skew, such as "at least one on-call"). Every participating transaction must use it.
- Escalate only for a named anomaly.

## Retries

- Retry serialization failures (40001) and deadlocks (40P01) by re-running the whole transaction, including its reads. Never re-run only the failed statement.
- Bound the attempts and use exponential backoff with jitter. Log retries.
- Retry only work that is idempotent or fully inside the transaction.

## Side effects and the outbox

Never call external systems (payments, email, queues, other services) inside a database transaction, or before its commit. A rollback or retry then duplicates, or orphans, the side effect. "Charge the card, then save the order" leaves a charged customer with no order when the save fails.

Use a transactional outbox. Write the event row in the same transaction as the state change, and let a separate relay publish it:

```sql
BEGIN;
INSERT INTO orders (tenant_id, id, status) VALUES ($1, $2, 'placed');
-- each statement binds its own parameters, numbered from $1
INSERT INTO outbox (tenant_id, topic, payload) VALUES ($1, 'order.placed', $2);
COMMIT;
```

Save-state-then-publish, and publish-then-save, are both dual writes. A crash between the two steps loses or duplicates the event.

## Idempotency

- Give each externally triggered write an idempotency key. Enforce it with a unique index, and claim it in the same transaction as the work:

```sql
INSERT INTO processed_request (tenant_id, key) VALUES ($1, $2) ON CONFLICT DO NOTHING RETURNING key;
```

  If no row comes back, the request was already handled. Look up the previous result instead of redoing the work.
- "Find by key, then act" is check-then-act. Two concurrent deliveries both pass the check.
- Handlers fed by at-least-once delivery (queues, webhooks, projections) must tolerate replays. An increment like `total = total + $x` double-counts on replay. Either:
  - guard it with a processed-event table, or a per-row `last_applied_version`, or
  - recompute from source.
- Save a consumer's checkpoint in the same transaction as the changes it covers. A checkpoint saved separately loses or repeats events after a crash, and a checkpoint advanced past a failed event drops that event forever.

## Polling a table for new rows

`WHERE id > $last_seen` on a sequence or identity column, or `created_at > $last_seen`, silently skips rows. Values are assigned at insert, but transactions commit in a different order. A slow transaction commits id 41 after the reader has moved past 42, and 41 is never read.

- Use a delivery mechanism that tracks commits: logical decoding, or a queue with `FOR UPDATE SKIP LOCKED` and a status column.
- If you must poll, only read rows older than a safe lag window (longer than your longest write transaction), and re-check on overlap.
