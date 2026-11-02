# Modules and tools

Read when touching go.mod, adding or upgrading a dependency, auditing vulnerabilities, or using gopls or golangci-lint. Checked on go1.27.1, gopls v0.24.0, golangci-lint 2.14.0 and govulncheck v1.3.0 unless marked.

## Modules
- `go.sum` holds checksums, not a lock. Versions come from `go.mod` through minimal version selection: the build uses the highest version any module requires, never the newest published one.
- `replace` and `exclude` take effect only in the main module. A library that ships a `replace` has no effect on its users.
- Tools used by the repo go in a `tool` directive (`go get -tool <pkg>`, run with `go tool <name>`) instead of a `tools.go` file with blank imports.
- Ask before adding a module. Before asking, check the stdlib and the modules already in go.mod.
- `go get -u ./...` upgrades everything at once and is not safe. Upgrade one dependency per change, read its changelog between the two versions, and run the tests.
- After any go.mod change: `go mod tidy`, then review the go.mod and go.sum diff (`review`'s dependency checks apply).

## Vulnerability check (govulncheck)
Run it before every handover of a Go change, and always after adding or upgrading a module or changing the `go` or `toolchain` line. Why: a dependency's advisory can land any day, not only when go.mod changes.

### Run it
```sh
govulncheck ./...                                          # installed binary, if it loads the module
go run golang.org/x/vuln/cmd/govulncheck@v1.3.0 ./...      # otherwise: built with the current toolchain
govulncheck -test ./...                                    # also scan code reached only from tests
govulncheck -scan module                                   # every advisory in go.mod, no patterns allowed
govulncheck -mode binary ./bin/app                         # the binary you actually ship
```
- An installed binary built with an older Go fails on a newer module with "package requires newer Go version ... (application built with go1.26)". Use the `go run` line; it fetches into the module cache and puts nothing on PATH. Say so once when it happens.
- Source mode checks the Go found on PATH, not the one CI or production builds with. Scan the release binary with `-mode binary`, or run with the same `GOTOOLCHAIN`, before calling a release clean.
- Pass the build tags the real build uses (`-tags integration,linux`). Other tags hide other code.
- Exit codes in text mode: 0 clean, 3 your code is affected. `go run` turns 3 into 1 (it prints `exit status 3`). `-format json`, `sarif` and `openvex` always exit 0, so a script must read the findings, not the exit code.
- It sends vuln.go.dev only module paths the database already lists, never code. Private module paths are not sent.

### Read the result
| Section | Means | Do |
|---|---|---|
| `=== Symbol Results ===` with a trace (`main.go:10: main calls language.Parse`) | your code calls the vulnerable function | a blocking finding: report it with the trace, the ID and the "Fixed in" version |
| "found N vulnerabilities in packages you import ... doesn't appear to call" | imported, not called | not blocking; list it in "Could improve" with the fix version (`-show verbose` lists them) |
| `-scan module` findings | the version in go.mod is affected, reachability unknown | use it for a quick audit, then confirm with a symbol scan |
| `Found in: stdlib@go1.27.1` | the Go toolchain itself | fixed by a newer patch release; propose raising the local Go and the `toolchain` line, never by editing code |

### Fix it
- Propose one upgrade per finding (module, current version, target at least "Fixed in", the changelog read between the two) and apply it after a yes: `go get <module>@<version>`, `go mod tidy`, the tests, then govulncheck again to show it gone.
- Never `go get -u ./...` to clear findings, and never add a `replace` to dodge one.
- If no fixed version exists, say so, show whether the vulnerable symbol is reachable, and let the user decide.

### Limits
- Only published advisories: a new malicious or typo-squatted package passes clean. `review`'s dependency checks cover that.
- Calls through `reflect` and `unsafe` are invisible (missed findings); interface and function-value calls are approximated (possible false ones). Binary mode can't show call traces.
- No way to silence a finding. Record an accepted risk in the PR or design doc, not in code.

## gopls
If gopls is reachable (MCP server, LSP tool or CLI), prefer it to grep for definitions, references, renames and diagnostics.
- Rename does not see struct tags, `text/template` or `html/template` names, or names used through `reflect`. Grep for those after a rename.
- `references` follows the build tags of the file you query, so code under other tags (`_windows.go`, `//go:build integration`) is missed.
- Call hierarchy covers static calls only, not calls through an interface.

## deadcode
`golang.org/x/tools/cmd/deadcode` is not part of Go; ask before installing it. From its own help (checked on the installed binary, 2026-11-02):
- It starts only from `main` packages. Add `-test` so code used only by tests counts as live.
- It handles interface calls, function values and reflection. It misses calls through `//go:linkname`.
- The result holds for one GOOS, GOARCH and build-tag set. Run it once per configuration the project ships.
- "Dead" is not "safe to delete": a dead method may still satisfy an interface. `-whylive=<pkg.Func>` shows the path that keeps a function alive.

## golangci-lint
- Use the repo's config as it is. Never add or widen one unasked; suggest it in "Could improve" instead.
- A suppression names the linter and gives a reason: `//nolint:errcheck // best-effort cleanup`.
- A v1 config needs `golangci-lint migrate` before v2 reads it.
- In a legacy tree, `issues.new-from-rev` limits findings to new code.

## Moving code
- Move a type between packages in two steps: add `type Old = new.Type` in the old place, move the callers, then delete the alias. Why: each commit builds and the diff stays reviewable.
- Keep the `=`. `type Old new.Type` declares a different type: it drops the methods, and callers stop compiling.
- Move a function the same way: leave `func Old() T { return new.Fn() }` with a `//go:fix inline` line in its doc comment. `go fix -inline -diff ./...` previews the rewritten callers; run it without `-diff` after a yes. Plain `go fix` runs every analyzer, so name `-inline`. (Checked on go1.27.1.)
- If an interface in the importing package can't break the cycle, move only the shared type to a small leaf package, or under `internal/` when it must stay private. It's a new package, so ask first.
- `gofmt -r 'pattern -> replacement'` rewrites by syntax with nothing to install. It doesn't know types, so a same-named call from another package matches too. Run it with `-d` first and read the diff.
- After a tool extracts or inlines code, read the diff. gopls Extract is reported to drop comments (not checked here).
- Break an import cycle with a small interface in the package that imports, not by merging packages.
- Keep a move or rename in its own commit, separate from any behaviour change.
