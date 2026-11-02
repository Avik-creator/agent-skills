---
name: which-skill
description: >-
  Picks the skill, or the short chain of skills, for a request that could match
  more than one or that needs several in order: a bug to diagnose and fix, a
  feature from spec to merge request, a release, a cleanup, a design. Use when
  it isn't clear which skill owns a request, or a request spans diagnosis,
  code, tests, review and shipping. Not needed when one skill plainly fits -
  load that one directly.
---

# Which skill

Route the request to the fewest skills that do it, then follow those skills. This skill holds no rules of its own about the work. Why: a rule copied here drifts from the skill that owns it.

## Scope

- **One skill:** when one fits, run it. A chain is for work that really has several stages, never to look thorough.
- **Two fit equally:** ask one question, then route.
- **Before routing:** ask every blocking question in one turn. Never invent a fact. Put an unknown under Open questions and say what it blocks. Replies are 1–2 lines; a paragraph only when the user asks.

## Pick

| The request | Skill |
|---|---|
| Write or change code, or a test that was asked for | `coding` |
| Write tests for a change, fix a flaky test, "what could break here" | `testing` |
| Manual test cases, QA coverage, how to test a flow by hand | `test-cases` |
| Find why something fails: a crash, wrong result, red CI, a production error | `debug` |
| Review a diff, a branch or a pull request, what a change could break, or a strict or thermo-nuclear quality review | `review` |
| Refactor, clean up, deduplicate, or standardize an area | `refactor` |
| Talk through an idea, a PRD, a spec, a system design, an architecture or scale review, a runbook, a README | `design-docs` |
| A throwaway prototype to answer one design question | `prototype` |
| Auth, input, uploads, secrets, a scanner finding, a threat model | `security` |
| Logs, metrics, traces, alerts, an SLO, a slow endpoint | `observability` |
| SQL, schema, migrations, transactions, pooling | `database` |
| Drive a real browser or app to check something | `automation` |
| Finish a change into a PR or MR, a rollback plan, a release checklist, a tag | `ship` |
| Write or update a PR or MR title or body, or draft review replies | `writing-pr` |
| Create or edit a skill | `writing-skills` |

**Alongside, not instead:**
- Load `go`, where installed, with any skill that touches `.go` files.
- Load `database` when the work reaches SQL.
- Load `security` when it crosses a trust boundary.
- Load `observability` when it adds a path on-call must see.

Read the chosen skill and follow it.

## Chains

Run the stages in order. Each skill's own stops still hold. Pause for the user's yes before any stage that writes code, pushes, or touches a shared system.

- **Bug:** `debug` finds and proves the cause, and stops. After a yes, `coding` fixes it, with the reproduction test written red first through `testing`. Then `review`. Add `test-cases` only when no automated test can reach it.
- **Feature:** `design-docs` when there is no agreed spec, then `coding`, then `testing`, then `review`. `ship` comes when the user wants it pushed.
- **Release:** `review`, then `ship`. Ship's done gate, its interview, and `writing-pr` for the description all run inside `ship`. A migration follows `database` and ship's deploy notes.
- **Cleanup:** `refactor`, which applies each approved step through `coding`.
- **Design:** `design-docs` for the system design, then a spec in `design-docs` if one is still needed, then `coding` after approval. `prototype` first when one question needs trying before deciding.
- **Slow:** `observability` measures it first. `database` for a slow query. Then `coding` for the change the measurement points to.
- **Live incident:** `debug`, reading production only. A mitigation is a proposal for the user. The fix then follows the bug chain.

## Limits that hold across chains

- `debug` and `refactor` change no code until the user says yes.
- `review` fixes proven findings only in the user's own work, and asks first before a schema, API, security or user-visible change. Someone else's PR is report-only.
- `ship` opens the PR or MR and stops. The user merges, deploys and tags.
- `design-docs` doesn't write code.
- No skill pushes to the default branch or writes to production.

## Replies

Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

Name the skill or chain in one line, then run the first stage.
