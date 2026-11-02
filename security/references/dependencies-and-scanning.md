# Dependencies and scanning

Read when triaging a finding, adding a dependency, preparing a release, or when the repo has no scanners. A new dependency is also reviewed with `review`'s `references/dependencies.md` (maintainer, age, typosquat, license, lockfile diff). Do not repeat that review here.

## Triage

```
Finding
├── Reachable on a runtime, build, test, or deploy path?
│   ├── yes → a fix is a candidate for this change
│   └── no, confirmed → still record it; it is not a blocker
├── Is a patched version published?
│   ├── yes → propose that upgrade, one finding at a time, after a yes
│   └── no → workaround, replace, or defer
└── Impact × likelihood, from threat-model.md
```

Apply one agreed upgrade after a yes, with the manager's normal command for that one package, and read the changelog between the two versions. A deferral names the reason and a review date. Do not run a bulk auto-fix (`npm audit fix --force`, `pnpm audit --fix`, `pip-audit --fix`, `cargo audit fix`) unless asked. Why: it upgrades packages nobody reviewed.

An audit matches published advisories. A new package, and a package whose name is one character off a known one, both come back clean.

## What to run

Run the scanners the project already configured (CI steps, and tools it already uses, such as semgrep, gosec, CodeQL, trivy, gitleaks). Then, on a security-relevant change, if the command is already available and you do not have to install it onto `PATH`:

| Project | Command |
|---|---|
| Go | govulncheck, as the go skill describes |
| npm | `npm audit` |
| pnpm | `pnpm audit` |
| Python | `pip-audit` |
| Rust | `cargo audit` |

Yarn's audit command was not re-checked. Use the command that version's own docs name.

`cargo audit` in a project with no `Cargo.lock` can generate one by running Cargo, and Cargo can run the project's code. On a project you do not trust, skip it and say so.

Also run the staged secrets check from SKILL.md. It prints names, never values.

Do not install a scanner, add one to CI, or add a config unless asked. A suppression names the rule and the reason. For semgrep the comment is `nosemgrep: <rule-id>` on the matched line or the line above; put the reason in that comment. `nostmt` is not the comment.

## Suggest, do not run

When the repo has no scanners, before a release, or when the user asks for a security audit, suggest this set. Run one only after a yes. Do not install it yourself.

| Tool | Command | Setup, after a yes |
|---|---|---|
| gitleaks | `gitleaks git --redact=100 .` | The current command is `git`, not the deprecated `detect`. `--redact=100` keeps the secret out of the output. Install a release binary from the gitleaks README. |
| semgrep | `semgrep scan --config auto` | Install the CLI from the Semgrep docs. `--config auto` pulls registry rules and the CLI docs say that reports pseudonymous metrics and the project URL. Add `--metrics=off` when the user does not want that. |
| osv-scanner | `osv-scanner scan source -r .` | Install a release binary from the osv-scanner docs. Do not `go install` an `@latest` build. |

Each finding they print goes through the triage above.

## Lockfile and install scripts

Checked on 2026-11-02. One lockfile per install root, committed, and CI does not rewrite it. If the manifest, the lockfile, and CI name different managers, stop and say so.

| Manager | Lockfile install | Scripts |
|---|---|---|
| npm 11.19.0 | `npm ci` requires `package-lock.json` or `npm-shrinkwrap.json`, errors if they disagree with `package.json`, and does not write either file | `--ignore-scripts` skips lifecycle scripts (default is false). `--strict-allow-scripts` (default false) fails the install when a dependency's install scripts are outside `allowScripts`, instead of warning |
| pnpm 11.11.0 | `--frozen-lockfile` fails if the lockfile would change; the help says this is already on in CI | `--ignore-scripts` skips lifecycle scripts |
| Yarn Berry | `yarn install --immutable` aborts if the lockfile would change (`yarnpkg.com/cli/install`) | The same page says `--mode=skip-build` skips build scripts for that install. Re-read the pinned version before relying on a default |

For a manager not in this table, read its current docs. Do not borrow another manager's flag.
