# Setting up a repo's verifier

Read the first time `prove` runs in a repo, or when the app gains a surface or a feature. The output is `.prove/verifier.md`, `.prove/features/` and `.prove/scripts/`, written for an agent that has never seen the app and reads it in the middle of another task: exact commands, real selectors, no placeholders.

## 1. Ask the repo, not the user
Answer these from the code, scripts, README and existing test tools. Ask the user only what you can't observe. Why: every question the repo could answer costs the user a turn.
- **Surfaces:** where users and programs meet the app: web pages, an API, a CLI or TUI, a desktop or mobile app, background jobs, emails, webhooks, an MCP server.
- **Run:** the repo's own start command, ports, environment variables, seed data and sign-in.
- **Drive:** existing harnesses first (Playwright or Cypress specs, API test helpers, PTY scripts). Only then a generic method from methods.md.
- **Observe:** what evidence each surface gives: screenshots, response bodies, exit codes, logs, rows.
- **Isolate:** can a second copy run beside the user's own (other port, other data folder)? If not, say so, and never drive the user's running copy.

If a surface can't be reached from this machine, propose a way in (a local emulator, a seeded test account, a sandbox, a local email catcher) and ask. Until then it is listed as not covered.

## 2. Show the plan, then write
Show one table and let the user pick the methods:

| Surface | Method | Answer key | Catches | Misses | Cost | Reachable now? |
|---|---|---|---|---|---|---|

Write only after a yes. Why: the methods decide what the verifier can ever catch.

## 3. Write `verifier.md`
Sections, each from what step 1 found:
- **Launch:** the exact command that starts the app from an empty data folder, migrations and seed data included, and how to tell it is ready to use. A process that answers is not ready; a database can pass a health check while its setup scripts still run.
- **Doctor:** one read-only check that this copy is worth driving: process up, right build, port owned by this run, sign-in valid. Run it first whenever anything looks off.
- **Drive:** one section per surface, with real commands and stable handles (ARIA labels, test ids, route paths, prompt strings), never screen positions.
- **Evidence:** what to capture and where it goes (`.prove/runs/<run>/`). The real user path, not internal setters or test-only endpoints. The action and the resulting state, plus side effects.
- **Cleanup:** stop what this run started, by process id or container name, never by process name. Never delete the evidence.
- **Scope map:** which source folders reach which surfaces and features, and the shared code (schema, auth, navigation, shared components) that reaches many.

Every helper in `.prove/scripts/` is executable, and `verifier.md` shows how to call it.

## 4. Seed the feature map
One file per user-facing feature in `.prove/features/`, starting with the top 3 to 5 (from routes, commands, menus or docs). Each has:
- **How to reach it,** as a user would.
- **Driving it,** with the method and real handles.
- **Done looks like:** the end state that proves it works, on screen and in stored data.
- **Gotchas:** what the seed data lacks, flaky steps, and what could fool the check.

A run that drives one convenient entry point is incomplete when the map lists others.

## 5. Prove the verifier once
- Follow `verifier.md` end to end on one feature: launch, doctor, drive, evidence, cleanup. Then check the evidence still exists.
- Break that feature on purpose in a scratch copy (a save that never happens, a count off by one, the right output with the wrong exit code). The verifier must say BROKEN. Undo the break and confirm a clean working tree.
- Run cleanup after every failed attempt, so no process or port is left behind.

A verifier nobody ran is a draft. A check that never failed proves nothing.

## When the app changes
A renamed button, a moved route or a new port makes the verifier describe last month's app. When a step fails for a reason unrelated to the change, decide which is wrong:
- **Drift** (the app changed on purpose): fix the step, prove the fix live, and tell the user what changed. This is the only way `verifier.md` changes during a build.
- **A product bug:** report it with evidence, and leave the step as it is.

Write any command that finally worked after a retry into `verifier.md` at once. Never edit an answer key here; that is the user's call.
