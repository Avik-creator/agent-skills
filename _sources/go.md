# Sources

Ideas were adapted and re-verified, not copied, except the bundled CLI wrapper. Pulled 2026-11-02.

| Upstream | License | Taken |
|---|---|---|
| JetBrains go-modern-guidelines (Claude plugin `modern-go-guidelines` 1.1.1, marketplace commit `40781f167719913666fe2a7dc1c77ea6f256df0a`, 2026-08-19) | Apache-2.0 | `scripts/run-tool.sh` and `scripts/VERSION` (CLI v0.1.1) copied unchanged; license in `scripts/LICENSE-modern-go-guidelines`. The skill's "CLI decides idioms" section follows its SKILL.md |
| samber/cc-skills-golang v2.0.1 (zip, no SHA recorded) | MIT | about 50 facts from 45 skills, each re-checked; see below |
| addyosmani/agent-skills (`debugging-and-error-recovery`, `api-and-interface-design`, `observability-and-instrumentation`, zip) | MIT | nothing yet; held for the debug, observability and API rounds |
| Author's working rules | n/a | stdlib first, wiring in main, no premature interfaces, one-line comments, repo conventions win, checks before handover, "Could improve" list |

## What was dropped from cc-skills-golang, and why
- 7 skills on the author's own libraries (samber/lo, mo, ro, do, oops, hot, slog), 4 DI-framework skills, cobra, viper, swagger, gqlgen, grpc, popular-libraries, stay-updated, how-to, pkg-go-dev (needs `godig`), CI templates, project-layout, refactoring workflow.
- Reasons: library manuals, conflicts with stdlib-first and one-line comments, behaviour the user forbids (writing `.modernize` files and agent-config lines unasked, committing per step, reading production through a SQL MCP), or facts already covered by the modern-go CLI.
- Wrong claims found there and not carried over (checked on go1.27.1 during review): `ConstantTimeCompare` safe for unequal lengths; `text/template` as XSS defence; `encoding/json` strict by default in 1.27; `for k := range slices.Sorted(...)` yields keys; pre-1.24 map internals; struct copies are "deep"; `httptest.NewTestServer(handler)` signature; a shutdown that hangs when the port is taken; a row iterator without `rows.Err()`; `math/rand` and `runtime.SetFinalizer` listed as deprecated; `b.Elapsed` valid only inside the loop.
- Its EVALUATIONS.md was not used as evidence: the author wrote the assertions, several grade preferences, and viper, cobra, fx and slog show 0-19 point gains.

## Verification
- Every code example in the references was compiled and run on go1.27.1 with `-race` (`gocheck3` scratch module): the shutdown exits on a bind error, `os.Root` rejects `..`, the synctest example passes and fails when the sleep is shortened.
- govulncheck v1.3.0 (via `go run`, toolchain go1.27.1) was run on scratch modules pinned to golang.org/x/text v0.3.5: calling `language.Parse` gave a Symbol finding (GO-2021-0113) and exit 3 (1 under `go run`); importing without calling gave exit 0 plus the "doesn't appear to call" note; `-scan module` rejected patterns and listed stdlib@go1.27.1 (fixed in 1.27.3); `-format json` exited 0; upgrading to v0.3.7 cleared the finding. The installed binary (built with go1.26) failed to load a `go 1.27` module, which is why the `go run` fallback exists. Privacy, build-config and limits text is from the v1.3.0 `cmd/govulncheck/doc.go`.
- Facts marked "not executed" or "not run here" in the references were read from source or docs only.

## Refresh
1. Bump the CLI: copy `run-tool.sh` and `VERSION` from the latest `JetBrains/go-modern-guidelines` plugin, check `list` still runs.
2. Bump the govulncheck pin in `references/modules-and-tools.md` to the latest `golang.org/x/vuln` tag and re-run the scratch modules.
3. On the newest Go: for each row in `references/toolchain.md`, run `go doc <symbol>` and grep `$(go env GOROOT)/api/go1.*.txt`; re-run the GODEBUG grep (`grep -n "Removed:" $(go env GOROOT)/src/internal/godebugs/table.go`); add the new release's section and update the "Checked on" line.
4. Re-run the scratch examples in traps.md, security.md and testing.md.

## Added 2026-11-02 (refactor sweep)
From cc-skills-golang `golang-refactoring` (structural.md:185, catalog.md:37,115, go-tooling.md:31-43,118-123, safety-net.md:27,32-33) and `golang-modernize`. Everything was checked on go1.27.1:
- **`type Old New` versus the alias:** a language fact.
- **The `//go:fix inline` forwarder:** tested in a scratch module. `go fix -inline -diff` rewrote `c.Old()` to `c.New()`.
- **`gofmt -r` / `-d`:** checked in `gofmt -h`.
- **deadcode:** from the installed binary's help, including `-test`, `-whylive`, the go:linkname gap, and one configuration per run. The source's claim that deadcode is unsound for reflection is **wrong**; its help says it handles reflection.
- **Coverage:**
  - Statement counting: a scratch run reported "% of statements".
  - `-coverpkg`: from `go help testflag`.
  - A package without tests reported `0.0% of statements`. The source's claim that such packages are left out is out of date.
- **Generic methods:** in the 1.27 spec (`doc/go_spec.html`: "a generic method [Go 1.27]"). golang-refactoring's "Go has no generic methods" is wrong for 1.27.
- **json/v2:** `JSONv2: true` is in the baseline in `internal/buildcfg/exp.go`, so `encoding/json/v2` is available by default while `encoding/json` keeps v1 behaviour, as toolchain.md already said. golang-modernize's "v2 is the default" is misleading.
- **Not checked:** that gopls Extract drops comments. The skill says so.
- **Dropped:** `eg` and `gopatch` (installs, and claims about them weren't checked), gopls Inline and package-rename details (not checked), and `// Deprecated` on forwarders (left to the repo's style).
These edits came after MR !3 (go 1.0.0), so the MR doesn't contain them.
