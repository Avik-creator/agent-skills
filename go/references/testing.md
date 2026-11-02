# Testing and benchmarks

Read when writing Go tests, examples, fuzz tests or benchmarks. Checked on go1.27.1 unless marked.

## Tests
- Table-driven with `t.Run`, fields named `got` / `want`, and a `name` per row that says the behaviour (`"refund after full ship is refused"`). Compare whole values (`reflect.DeepEqual`, or `cmp.Diff` if the repo already uses go-cmp) so a missing field fails.
- Test through the package's exported behaviour. Why: tests that reach into internals break on every refactor.
- Use the standard library unless the repo already uses testify. If it does: `require` for preconditions, `assert.ErrorIs` for wrapped errors, and build `assert.New(t)` inside each subtest (one built in the parent reports subtest failures on the parent).
- `t.Helper()` in helpers, `t.Cleanup` instead of defer chains, `t.TempDir()` for files, `t.Setenv` for env (it can't be combined with `t.Parallel`).
- Add `t.Parallel()` only to tests that share nothing. Why: shared fixtures under parallel tests give order-dependent failures; `-shuffle=on` exposes them.
- Integration tests run against the real engine (a throwaway container), never a shared or production database. See the database skill.
- A fuzz test (`FuzzXxx` with `f.Add` seeds) is worth it for parsers and decoders of untrusted input. Commit the `testdata/fuzz` cases it finds.

## Time and concurrency in tests
- Use `testing/synctest` instead of sleeps or a fake-clock library:
  ```go
  synctest.Test(t, func(t *testing.T) {
  	c := newCache(time.Minute)
  	c.Set("k", 1)
  	time.Sleep(61 * time.Second) // fake clock: returns instantly
  	synctest.Wait()              // let the cache's goroutines settle
  	if _, ok := c.Get("k"); ok {
  		t.Fatal("entry did not expire")
  	}
  })
  ```
  The clock only advances when every goroutine in the bubble is durably blocked; a goroutine blocked on a mutex or real I/O doesn't count. Inside the bubble, don't call `t.Run`, `t.Parallel` or `t.Deadline`. `synctest.Sleep(d)` is `time.Sleep(d)` plus `synctest.Wait()`.
- `httptest.NewTestServer(t, h)` serves over an in-memory network and works inside a synctest bubble. Calling `Start` moves it to a real loopback socket. It registers its own cleanup.
- Run with `-race` locally for any code with goroutines; CI without `-race` misses data races entirely.

## Examples
- `func ExampleStore_Get()` documents method `Store.Get`. A suffix after a further `_` must start lowercase: `ExampleStore_Get_cached`. `ExampleStore_Get_Cached` fails `go vet` ("malformed example suffix").
- An `// Output:` comment makes the example a test.

## Benchmarks
- Write the loop exactly as `for b.Loop() { ... }` and don't also loop to `b.N`. Setup before the loop isn't timed. Only calls inside the braces are kept from being optimised away. After the loop, `b.N` and `b.Elapsed()` are valid for custom metrics.
- `go test -bench=.` still runs every test first. Add `-run='^$'` to run only benchmarks.
- Compare with `benchstat` over repeated runs (`-count=10`), run one benchmark set at a time on a quiet machine, and don't claim a change where benchstat shows `~`. `benchstat` is `golang.org/x/perf/cmd/benchstat`; ask before installing it.
- Compare only on the same Go version. See toolchain.md for releases that moved allocation and GC baselines.
- `-benchmem` for allocations; `-cpuprofile` / `-memprofile` and `go tool pprof` to find where time goes before changing code.
- `go build -gcflags=-m` shows escape and inlining decisions for an unexpected heap allocation.

## Coverage
- Go counts statements, not branches. A covered `if` line says nothing about the branch that was never taken, so read the branches of the lines you change.
- By default a test measures only its own package. Add `-coverpkg=./...` to count lines it reaches in other packages.
- Packages with no test files are listed at `coverage: 0.0% of statements` (checked on go1.27.1), so a repo total includes them.
