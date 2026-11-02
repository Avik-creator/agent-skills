---
name: go
description: Writes, changes and reviews Go code to current standards - version-correct idioms from the Modern Go Guidelines CLI, stdlib-first boring design, and a fixed set of checks before handover - plus verified references for the traps models still miss (typed nil, slice aliasing, context and goroutine lifetimes, json v1 vs v2), stdlib security (os.Root, crypto/rand, CSRF, server timeouts), testing and benchmarks (table tests, b.Loop, synctest), modules, and dated toolchain changes. Use when writing, editing, fixing, refactoring, testing, benchmarking or reviewing any .go file, go.mod or go.work, upgrading the Go version, adding a Go dependency, or when a Go build, vet or test fails, even if the user never says "Go". Not for SQL, schema or pgx query design - use database; not for reviewing a whole diff against a spec - use `review`.
---

# Go

Write Go the way the project's Go version and conventions expect, keep it boring, and prove it with the project's own checks. Every rule carries its reason; apply the reason to cases the rule doesn't name.

## Quick start
1. Read go.mod (`go` line, toolchain) and the repo's conventions: lint config, Makefile or CI targets, two or three files near the change.
2. Run the modern-go `list` for the file you will edit and read all of it.
3. Write the tests first and watch them fail, then the code.
4. Run the checks, then hand over with what ran, what didn't, and the "Could improve" list.

## Modern idioms: the CLI decides
Run it before editing a Go file. It reads the Go version from go.mod, so it never suggests an API the module can't use.
```sh
sh <skill-dir>/scripts/run-tool.sh list --file-path path/to/file.go
sh <skill-dir>/scripts/run-tool.sh explain errors_as_type sync_waitgroup_go   # only the IDs you will apply or skip
```
- If the `use-modern-go` plugin is installed, use its script instead; it may be newer. The bundled copy is pinned to the version in `scripts/VERSION`.
- The first run installs the CLI with `go install` into `~/.cache/go-modern-guidelines/`. Say so once when it happens.
- Read the whole `list` output; never pipe it through `head` or `grep`. Why: it is ordered newest first and older entries still apply.
- Apply a returned guideline to the code you are editing, not to the rest of the file. Skip it only when it wouldn't compile, would change behaviour, or doesn't match; run `explain` before skipping one that looks relevant.

## Defaults (the repo's documented conventions win)
When the repo already does it differently, follow the repo and mention the difference once at handover. Why: a skill that fights the codebase produces diffs nobody can merge.
- **Stdlib first.** Ask before adding a module. Why: every dependency is code you ship and must keep patched.
- **Wire by hand in `main`**: config, then infrastructure, then repositories, then services, then transport. No DI framework, service locator, globals or `init()` for setup. Why: the startup order stays readable and a missing dependency fails at compile time.
- **No interface until a second implementation or a test needs one**, and define it where it is used. Why: interfaces made up front freeze the wrong shape.
- **Plain constructors with parameters or a config struct**; functional options only for a public library with many optional knobs. Why: options hide required inputs.
- **One-line comments**: say what the symbol is or does; two lines only for a real trap. Doc comments start with the name. Why: the why belongs in the commit or PR.
- **Errors**: wrap with context (`fmt.Errorf("load user: %w", err)`), log or return but not both, keep messages static and put IDs in slog attributes. Why: dynamic messages break grouping in log and error tools.

## Checks before handover
Use the project's own targets first (`make check`, Taskfile, CI workflow steps). Why: that is what CI will judge the work by. When there are none, run this baseline:
```sh
test -z "$(gofmt -l .)"
go vet ./...
go test -race -shuffle=on ./...
go mod tidy && git diff --exit-code go.mod go.sum
```
- `golangci-lint run` only if the repo has a golangci config. Never add one unasked.
- `govulncheck ./...` on every handover, even when the project has its own targets: a called vulnerability blocks, an uncalled one goes in "Could improve". Running it when the installed binary is too old, reading and fixing findings: [modules-and-tools.md](references/modules-and-tools.md#vulnerability-check-govulncheck).
- Install nothing (the `go run` fallbacks only fill the module cache) and change no CI or config to make a check pass.
- Watch each new test fail once before the fix. Why: a test that can't fail proves nothing.

## Hand over
- What ran and the result; a baseline check the project lacks goes in "Could improve", not into this run.
- What you didn't verify, and any repo convention you followed instead of a default.
- **Could improve**: a short list of gaps found, each with the one-line fix, applied only after a yes. Examples: CI runs tests without `-race`; no lint config; CI never runs govulncheck; the `go` line is old and the CLI shows idioms that a newer one unlocks; a dependency with a stdlib replacement.

## Replies

Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

## References
Read only the file the task needs.

| File | Read when |
|---|---|
| [traps.md](references/traps.md) | writing or reviewing any non-trivial Go: nil, slices, maps, errors, context, goroutines, JSON, database/sql, shutdown |
| [security.md](references/security.md) | user input, files, paths, tokens, crypto, cookies, CSRF, HTTP servers, error responses |
| [testing.md](references/testing.md) | writing tests, examples, fuzz or benchmarks, timing-dependent tests, testify in an existing repo, reading coverage |
| [modules-and-tools.md](references/modules-and-tools.md) | go.mod, adding or upgrading a dependency, vulnerability checks, gopls, golangci-lint, deadcode, moving or renaming code between packages |
| [toolchain.md](references/toolchain.md) | upgrading the `go` line or toolchain, GODEBUG, or using an API from a recent release |
