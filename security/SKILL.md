---
name: security
description: Threat-models a change that crosses a trust boundary, turns each abuse case into a failing test, triages a scanner or advisory finding, and writes a design review into the design doc. Use when adding or changing auth, login, sessions, cookies, passwords, tokens, JWT, permissions, roles, user input, file uploads, webhooks, a fetch of a user-supplied URL, secrets, API keys, CORS, CSRF, XSS, SQL injection, a vulnerability, a CVE, a dependency or npm audit finding, a threat model, personal data, PII, GDPR, or an LLM feature that handles untrusted text, or when the user asks whether something is secure. Not for reviewing a whole diff - use `review`; not for Go-specific APIs - use go; not for SQL or schema design - use database.
---

# Security

Treat a value as trusted only because of who wrote it. Write the threat model and the abuse-case tests before the feature. Leave the result in chat, or as a section of the design doc. Do not create a security-output folder.

## Quick start

1. Pick a mode: building across a trust boundary, triaging a finding, or reviewing a design.
2. Stop for a yes on anything in the ask-first list.
3. Run the checks the project already has, plus the two checks under Scanners.
4. Hand over what ran, what did not, and a "Could improve" list.

## Building across a trust boundary

Use this when the change accepts input, decides who may act, sets a session, takes an upload, fetches a URL, reads a secret, opens a file named by data, or handles model output.

1. In chat, name the trust boundaries, the assets, the STRIDE letters for each interaction, and 2–5 abuse cases. Trust follows who wrote a value, not which channel delivered it. Read [threat-model.md](references/threat-model.md).
2. Turn each abuse case into a test and watch it fail, then write the code. Why: a control you have not seen fail is unproven.
3. Apply the controls in [controls.md](references/controls.md). A model feature also needs [llm.md](references/llm.md).
4. Personal, payment, or health data also needs [privacy.md](references/privacy.md).

**Example.** "Fetch the title of a URL the user pastes." Caller to process: all of STRIDE. Process to outbound call: tampering, information disclosure, denial of service, and the caller picks the destination. One abuse case: the URL resolves to a link-local address. The test expects that fetch to be refused, and it fails before the check exists.

## Triage a finding

A scanner hit, an advisory, or a reported bug. For each one, say whether the code is on a runtime, build, test, or deploy path, whether a fix is published, and which cell of the impact × likelihood grid it sits in. Propose one fix and apply it after a yes. A deferral records a reason and a review date. Do not run a forced remediation (`npm audit fix --force` or the equivalent). Why: it can change dependency ranges the project did not choose. Steps: [dependencies-and-scanning.md](references/dependencies-and-scanning.md).

## Design review

Write the STRIDE table, tag each control preventive, detective, or corrective, and write the residual risk. Add that section to the design doc through the design-docs skill, using the template in [threat-model.md](references/threat-model.md). Why: a review that exists only in chat disappears.

## Ask first

Stop for a yes before a new or changed auth flow, a new category of personal or sensitive data, a CORS change, an upload handler, a rate-limit change, granting a role or permission, or a new external integration. Why: each of these is hard to take back once it is live.

## Scanners

Run the scanners the project already configured. On a security-relevant change, also run the ecosystem's known-vulnerability check when that command is already available (govulncheck as the go skill describes, `npm audit`, `pnpm audit`, `pip-audit`, or `cargo audit`), and this secrets check, which prints file names only:

```sh
git diff --cached -i -G'password|secret|api_key|token' --name-only
```

A printed name is a file to open before you commit. Why: the check must not copy a secret into the tool log. Do not install a scanner, add one to CI, add a config, or suppress a finding unless asked. A suppression names the rule and the reason. Commands and the optional scanner set: [dependencies-and-scanning.md](references/dependencies-and-scanning.md).

## Hand over

- What ran, and the result. What you could not run, and why.
- The threat model, the triage, or the design-doc section.
- **Could improve:** each gap with a one-line setup. Propose it, and apply it after a yes.

## References

| File | Read when |
|---|---|
| [threat-model.md](references/threat-model.md) | starting a feature, rating a finding, or writing the design-review section |
| [controls.md](references/controls.md) | implementing a control |
| [llm.md](references/llm.md) | the feature calls a model or puts untrusted text in a prompt |
| [dependencies-and-scanning.md](references/dependencies-and-scanning.md) | a finding, a new dependency, a release, no scanners, or choosing the audit command |
| [privacy.md](references/privacy.md) | personal, payment, or health data |
