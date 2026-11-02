# Traps

Read when writing or reviewing non-trivial Go. These are the mistakes that compile, pass a quick test, and fail later. Each was checked on go1.27.1 unless marked.

## Nil
- **A typed nil in an interface is not nil.** A function returning `error` that returns a nil `*MyErr` gives a non-nil error, and the caller's `if err != nil` fires. Return a literal `nil` on success.
  ```go
  func get() error { var e *MyErr; return e }   // bad: get() != nil is true
  func get() error { return nil }               // good
  ```
- Nil values: reading a nil map is fine, writing panics; `append` to a nil slice is fine; send or receive on a nil channel blocks forever; `close` of a nil channel panics.
- `any` or interface map keys panic at runtime when the dynamic type is unhashable (`m[any([]int{1})]`: "hash of unhashable type").

## Slices and maps
- **`append` can write into the caller's array.** `b := a[:2]; append(b, x)` overwrites `a[2]` when there is spare capacity. Cap it first with `a[:2:2]`, or `slices.Clone`, before appending to a slice you were given.
- A small subslice of a large buffer keeps the whole buffer alive. `slices.Clone` it before storing it.
- A getter that returns an internal slice or map hands out mutable state. Return `slices.Clone` / `maps.Clone`.
- Struct and array copies are shallow: a copied struct still shares its slices, maps and pointers.
- `for k := range slices.Sorted(maps.Keys(m))` ranges over a slice, so `k` is the index. Use `for _, k := range ...`.
- Map iteration order is random by design; sort the keys when output order matters.

## Errors
- Wrap with `%w`; `fmt.Errorf` takes several `%w` verbs. Match with `errors.Is` / `errors.As` (the CLI adds `errors.AsType` where the version allows).
- Use `%v` instead of `%w` at a public package boundary when callers must not depend on the inner error type.
- `errors.AsType` only accepts error types; use `errors.As` to test for a non-error interface.
- Log or return, not both: logging at every layer prints one failure five times. The boundary that handles it logs once.
- Keep the message static and low-cardinality (`"load user"`), and put IDs, paths and counts in slog attributes or the wrap chain.
- Never send `err.Error()` to a client: it leaks driver, path and SQL detail. Map to a fixed response and log the cause.

## Context
- Pass `ctx` as the first parameter; never store it in a struct. Use the `*Context` variants (`QueryContext`, `http.NewRequestWithContext`).
- Work that must outlive the request uses `context.WithoutCancel(ctx)`, which also drops the deadline (verified: `Deadline()` returns `ok == false`). Add your own `WithTimeout`, or the work can run forever.
- **singleflight shares the first caller's call.** If that caller's ctx is cancelled, every waiter gets the error. Run the shared function on `context.WithoutCancel(ctx)` plus a timeout.
- Every `WithCancel` / `WithTimeout` gets its `cancel` called, normally `defer cancel()`. Why: otherwise the timer and child context leak until the parent ends.

## Goroutines
- Every goroutine needs a known way to stop (ctx, closed channel, or bounded work). Why: a leaked goroutine holds memory and connections forever.
- A panic in any goroutine kills the process. Neither `wg.Go` ("f must not panic") nor `errgroup` recovers it (x/sync v0.23.0 refuses on purpose). Recover at the goroutine's top only where the process must survive a bad item, and turn it into an error.
- Bound fan-out with `errgroup.SetLimit(n)` instead of a hand-rolled worker pool.
- Go 1.23+ timers: the channel is unbuffered, `Reset` and `Stop` need no drain. Still avoid `time.After` inside a hot `select` loop; reuse one timer.
- A mutex guards data, not code: keep the critical section small and never hold it across I/O or a channel send.
- `sync.Pool`: don't return very large buffers to it; they pin memory. Drop buffers above a size cap.

## JSON
- `omitempty` does not omit a zero struct or `time.Time` (`{"sub":{"A":0}}`, `"0001-01-01T00:00:00Z"`). Use `omitzero`.
- `encoding/json` keeps v1 behaviour even with the v2 experiment: duplicate keys and case-insensitive names are accepted. Only `encoding/json/v2` is strict. Don't "fix" v1 code for v2 rules unless you migrate to v2 on purpose.
- Decoding untrusted bodies: cap them with `http.MaxBytesReader`; use `DisallowUnknownFields` only where rejecting extra fields is the contract.

## database/sql
Full query, locking and pgx rules live in the database skill. The Go-side traps:
- `defer rows.Close()` right after the error check, and check `rows.Err()` after the loop. Why: Close releases the connection, and a mid-stream error otherwise looks like a short result.
- `QueryRow` never returns `sql.ErrNoRows`; `Scan` does.
- `Exec` for statements that return no rows; `Query` there leaks a connection until Close.
- A row iterator (`iter.Seq2`) must yield the `rows.Err()` at the end, or it silently truncates.

## HTTP servers and shutdown
- Set `ReadHeaderTimeout` (slowloris). A short `ReadTimeout` or `WriteTimeout` also cuts off large bodies and streaming responses; set them per need.
- `http.DefaultMaxIdleConnsPerHost` is 2. A client calling one host at high concurrency needs its own `Transport` with a higher `MaxIdleConnsPerHost`.
- Always close the response body, including on error status codes.
- Graceful shutdown must also exit when `ListenAndServe` fails at startup (port in use), not only on a signal:
  ```go
  ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
  defer stop()
  errc := make(chan error, 1)
  go func() { errc <- srv.ListenAndServe() }()
  select {
  case err := <-errc:
  	return err
  case <-ctx.Done():
  }
  sctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
  defer cancel()
  return srv.Shutdown(sctx)
  ```
- `os.Exit` and `log.Fatal` skip deferred calls. Return errors to `main` and exit there once.
- Check `Close` and `Flush` errors on write paths (files, buffered writers, gzip). Why: that is where the write actually fails.

## Types
- In a type switch, list concrete types before interfaces they implement; a `case io.Reader` above `case *os.File` makes the second unreachable, and vet stays silent.
- Embed a `noCopy` field (Lock/Unlock methods) in types that must not be copied; `go vet` copylocks then flags copies.
- Enums from `iota`: make 0 an explicit `Unknown`, so an unset value is never a valid state.
