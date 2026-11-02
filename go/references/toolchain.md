# Toolchain changes (dated)

Read when upgrading the `go` line or toolchain, setting GODEBUG, or using an API from a recent release. Idioms are not here; the modern-go CLI owns those.

**Checked on go1.27.1, 2026-11-02**, against `$GOROOT/api`, `go doc` and the toolchain source. Before relying on a note, confirm it with `go doc` or the source for the project's own version. Notes go stale; refresh steps are in `_sources/go.md`.

## Which release added it
| API or behaviour | Since |
|---|---|
| `os.Root` / `os.OpenRoot`, `crypto/rand.Text`, `tool` directive in go.mod | 1.24 |
| `testing/synctest.Test`, `http.NewCrossOriginProtection`, `runtime/trace.NewFlightRecorder`, container-aware `GOMAXPROCS` | 1.25 |
| `slog.NewMultiHandler`, `errors.AsType` | 1.26 |
| `uuid` package, `encoding/json/v2`, `httptest.NewTestServer(t, h)`, `synctest.Sleep`, generic methods (spec: "a generic method [Go 1.27]") | 1.27 |

A module whose `go` line is older than the release can't use the API: `go test` runs the `stdversion` vet check and fails the build (verified at a `go 1.21` line). It was not in `go test`'s vet set in 1.25.7; it is in 1.27.1.

## Go 1.27
- **Removed GODEBUG settings.** Pinning one to its old value in go.mod `godebug`, `//go:debug` or the environment now fails: `asynctimerchan`, `gotypesalias`, `x509keypairleaf`, `tls10server`, `tls3des`, `tlsrsakex`, `tlsunsafeekm` (`internal/godebugs/table.go`, `Removed: 27`). Delete the pin and fix the code that needed it.
- **`tracebacklabels` now defaults to 1**: goroutine pprof labels appear in crash tracebacks. Set `GODEBUG=tracebacklabels=0` if labels can hold personal data.
- **`encoding/json` stays v1** in behaviour; `encoding/json/v2` is the new strict package (duplicate keys rejected, case-sensitive names). Migrating is a decision, not a cleanup.
- **`uuid`** (import path `uuid`): `New`, `NewV4`, `NewV7`, `Parse`, `MustParse`. No v3 or v5. Prefer it to a third-party UUID module for new code.
- **`runtime/pprof` `goroutineleak` profile** is registered: use it to find goroutines blocked forever.
- **`go mod tidy` at `go 1.27`** keeps requirements in at most two `require` blocks (`SimplifyRequireVersion = "1.27"`; not executed on a real module).
- **Benchmarks:** allocation and GC changes (size-specialised malloc under 81 bytes, off under `-race`; the Green Tea GC on by default, from `internal/buildcfg`) move baselines. Never compare benchmark numbers across 1.26 and 1.27; the size of the shift was not measured.
- **Maps** have been Swiss tables since 1.24. Bucket and load-factor descriptions from older material are wrong.
- The compiler's base inline budget is still 80 (`inlineMaxBudget`); don't quote a higher number.

## Go 1.25
- `GOMAXPROCS` follows the container CPU limit, so `go.uber.org/automaxprocs` can be removed once the `go` line is 1.25 or later.
- Flight recorder: `trace.NewFlightRecorder(trace.FlightRecorderConfig{MinAge: ..., MaxBytes: ...})`, `Start`, then `WriteTo` when something goes wrong. Only one can be active, and `MaxBytes` is a hint, not a cap.

## Go 1.24
- `os.Root` for any file path that comes from a user (see security.md).
- `go get -tool <pkg>` adds a `tool` line; run it with `go tool <name>`.
