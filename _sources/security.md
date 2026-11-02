# Sources

Ideas were adapted and re-checked, not copied. Pulled 2026-11-02.

| Upstream | License | Taken |
|---|---|---|
| addyosmani/agent-skills `skills/security-and-hardening` (SKILL.md, `references/hardening-patterns.md`) and `references/security-checklist.md` | MIT, Copyright (c) 2025 Addy Osmani | Trust follows who wrote a value; STRIDE as a lens; abuse cases before code; ask-first list; allowlists, parameterized queries, output encoding, per-object authorization, cookie flags, CORS not `*` with credentials, generic errors, SSRF (all records, private/reserved, no redirects, DNS-rebinding gap), destructive-path containment and its two limits, shared-store rate limits, rotate-then-purge, the staged `git diff --cached -i -G... --name-only` check, audits match published advisories only, no forced remediation, deferrals with a reason and a review date, field classification, retention, deletion past backups and caches, vendor sharing is sharing, model output is untrusted input |
| addyosmani/agent-skills `references/security-checklist.md` install-script section | MIT, same | The idea of one lockfile and scripts blocked before the first install. The version matrix was not copied; the rows below were re-checked |
| Seth Hobson agents-main `plugins/security-scanning/skills/stride-analysis-patterns/references/details.md` Template 4 | MIT, Copyright (c) 2024 Seth Hobson | Which STRIDE letters apply to external→process, process→datastore, and process→process |
| agents-main `plugins/security-scanning/skills/threat-mitigation-mapping` | MIT, same | The three control types: preventive, detective, corrective. The scoring code was not used |
| agents-main `plugins/backend-api-security/agents/backend-security-coder.md` | MIT, same | Log injection and session fixation, as two controls. Nothing else from that agent |
| agents-main `plugins/security-compliance/commands/compliance-check.md` | MIT, same | The list of regimes to raise as a question (GDPR, CCPA, HIPAA, PCI DSS). No control code, no policy text |
| Author's library rules | n/a | Output in chat or the design doc, three modes, ask-first, "Could improve", point at code-review, go, database, design-docs |

## What was dropped, and why

- Path containment via `startswith` on a resolved string (`security-sast.md`). A root of `/var/www/uploads` also matches `/var/www/uploads2`. The skill uses a relative-path check after symlink resolution, plus the two limits already in the checklist.
- `shlex.quote` on arguments that are already a list (`security-sast.md`). Quoting is for a shell string. The skill does not show it.
- A trailing `|| true` on audit commands, `govulncheck@latest`, counting the word CRITICAL by grepping JSON, auto-applying patch upgrades, and the invented priority-score formula (`security-dependencies.md`). Triage is reachability, a published fix, and a named impact × likelihood cell. No score.
- `nostmt` as a Semgrep suppression (`sast-configuration`). The documented comment is `nosemgrep`. Checked at `https://docs.semgrep.dev/ignoring-files-folders-code` on 2026-11-02.
- `returntocorp/semgrep-action`. The repository README says the project is deprecated. The skill does not add a scanner to CI unless asked, so no replacement workflow is included.
- `safety check`. Not re-checked, and not named. Python's check in this skill is `pip-audit`.
- The upstream install-script matrix (npm 11.18 vs 12, pnpm 10.1–10.25 vs 10.26 vs 11, Yarn 4.14 vs earlier). Not re-checked. Only the commands verified below are written down.
- Universal numbers from the upstream skill: bcrypt "≥12", auth "10 per 15 minutes", 5 MB uploads, 24-hour cookies, HSTS `max-age=31536000`, `X-XSS-Protection: 0`. Not re-verified, so not used. Password parameters are the OWASP rows below.
- "Security retrofitting is 10x harder." An unmeasured claim. Not used.
- Attack-tree construction, security-requirement extraction, and the rest of sast-configuration (SonarQube, CodeQL setup, custom rules). Not in the decisions.
- Compliance-check's Python classes, privacy-policy generator, SOC 2, and PCI transaction-count levels. The skill must not write legal text or say "compliant".
- Product names from the upstream examples (frameworks, vendors, cloud products). The library is generic.

## Verified external facts

Checked on 2026-11-02.

| Fact | Where checked |
|---|---|
| Argon2id minimum rows (m/t/p), scrypt rows, bcrypt work factor ≥ 10 and 72-byte input, PBKDF2-HMAC-SHA-256 ≥ 600,000 when FIPS-140 is required | https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html |
| GDPR Article 3 territorial scope, and Article 33: controller notifies without undue delay and, where feasible, not later than 72 hours after becoming aware, unless the breach is unlikely to risk rights and freedoms; a late notice states why; the processor notifies the controller without undue delay | https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=celex%3A32016R0679 |
| PCI DSS: card verification codes are sensitive authentication data and requirement 3.3.1.2 prohibits storing them after authorization, even encrypted | https://www.pcisecuritystandards.org/faqs/are-merchants-allowed-to-request-card-verification-codes-values-from-cardholders/ |
| PCI DSS v4.0.1 requirement 3.3.1.3: PIN and PIN block are not stored after authorization. Read from a copy of SAQ B (document says PCI SSC, October 2024), not from the Council's own host | https://treasurer.nebraska.gov/tm/pci-dss/documents/PCI_DSS_v4_0_1_SAQ_B.pdf |
| PCI DSS v4.0.1 scope sentence: intended for entities that store, process, or transmit cardholder data and/or sensitive authentication data. Read from a copy of the standard | https://commerce.uwo.ca/pdf/PCI-DSS-v4_0_1-2.pdf |
| `semgrep scan --config auto` (also written `--config=auto`) runs the default ruleset. Registry use reports pseudonymous metrics; `--metrics=off` turns that off | https://docs.semgrep.dev/running-rules and https://docs.semgrep.dev/cli-reference |
| `nosemgrep` / `nosemgrep: <rule-id>` on the match line or the line above | https://docs.semgrep.dev/ignoring-files-folders-code |
| `gitleaks git` scans a repo (`detect` is deprecated and hidden). `--redact` takes a percent, default 100, so the skill writes `--redact=100` | https://github.com/gitleaks/gitleaks |
| `osv-scanner scan source -r <dir>` scans lockfiles recursively. `scan source` is the default. Install docs also show `go install ...@latest`; the skill does not | https://google.github.io/osv-scanner/usage/scan-source |
| `pip-audit` audits the environment or a project path. `--fix` rewrites dependencies, so the skill forbids it unless asked | https://github.com/pypa/pip-audit |
| `cargo audit` at the project root. With no `Cargo.lock` it may run Cargo, which can execute project code | https://github.com/rustsec/rustsec/blob/main/cargo-audit/README.md |
| npm 11.19.0, local `npm help ci`: `npm ci` requires a lockfile, errors if it disagrees with `package.json`, and does not write the lock. `--ignore-scripts` default false. `--strict-allow-scripts` default false; when true, install scripts outside `allowScripts` fail the install | local npm 11.19.0 help |
| pnpm 11.11.0, local help: `--frozen-lockfile` fails if the lockfile would change and is on by default in CI. `--ignore-scripts` skips lifecycle scripts. `pnpm audit` exists | local pnpm 11.11.0 help |
| `yarn install --immutable` aborts if the lockfile would change. `--mode=skip-build` skips build scripts for that run | https://yarnpkg.com/cli/install |
| CSRF origin checks: reject `Sec-Fetch-Site: cross-site` for unsafe methods, else match `Origin`, else `Referer`; missing headers and `Origin: null` are not same-origin; reject unless a token passes | https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html (checked by the reviewer) |
| `git diff --cached -i -G'password\|secret\|api_key\|token' --name-only` is accepted by git 2.54.0 (Apple Git-157). On a staged file containing `Password`, `-i` prints the file name and the same command without `-i` prints nothing. `--name-only` does not print the line | local git 2.54.0 |

## Not verified, so left out

- Yarn's audit subcommand for the installed major. The skill says to use that version's own docs.
- CCPA's current tests and thresholds. The skill tells the agent to ask, not to decide.
- HIPAA's covered-entity definition from the statute. Same: ask, don't decide.
- `safety check` as a current command.
- HSTS max-age, `X-XSS-Protection`, and any universal rate-limit or upload-size number.
- The cloud metadata address as a cited value. The control says link-local and the other reserved ranges, without an address.

## Refresh

1. Re-read the OWASP Password Storage Cheat Sheet and replace the parameter table if the rows moved.
2. Re-read GDPR Articles 3 and 33 on EUR-Lex before changing the 72-hour sentence.
3. Re-run `npm help ci`, `pnpm install --help`, and the Yarn install page, and update the "Checked on" line and the version numbers.
4. Re-read the gitleaks, semgrep, and osv-scanner command pages before changing a suggested command. `gitleaks detect` stays out unless the project docs revive it.

## Added 2026-11-02 (source sweep)
From wshobson/agents `frontend-mobile-security` (`xss-scan.md`, `frontend-security-coder.md`, `mobile-security-coder.md`), `deployment-validation/config-validate.md`, `cicd-automation` (read by a Cursor subagent): browser output rules (`textContent`, a sanitizer, http/https URLs only), webhook signature over raw bytes, PKCE and subresource integrity, WebView defaults, refusing a production boot in debug mode, short-lived CI credentials. PKCE checked: RFC 9700 §2.1.1 "Public clients MUST use PKCE" (https://www.rfc-editor.org/rfc/rfc9700.html). Dropped: always pinning certificates (ask-first), client checks as a control, PBKDF2 at 100000 with a fixed salt, `rel=noopener` (browsers now imply it; not re-checked).
