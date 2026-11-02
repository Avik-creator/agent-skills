# deliver: sources

Written 2026-11-03. Ideas adapted, not copied.

## Ray Amjad: ray-amjad/skills (MIT, Copyright (c) 2026 Ray Amjad)
Read in full from the downloaded `skills-main` folder: `idea-to-spec` and its references, `task-lifecycle` and its references, and the README.
- **Taken:**
  - do the homework before question 1, and never ask the user a fact the environment can answer
  - one decision per turn with a recommendation, then one playback of the whole plan before anything is written; a re-played part after any change
  - numbered defaults the user can veto, and the choices left to the agent, listed in the playback
  - the chain: the spec's proof is the answer key, the builder builds to it, verify drives the real app, and BROKEN goes back to the builder, which fixes the app, never the key
  - stop and ask before merging; the user merges
- **Dropped:**
  - publishing the spec as a GitHub issue, the fresh-agent critique of the draft, and the invariants and ADR workflow
  - `task-lifecycle`'s builder, reviewer and fixer agents and its merge step; this library's `coding`, `review` and `ship` do those parts
  - the course links and the beta notes

## pstack, in cursor/plugins (MIT, Copyright (c) 2026 Lauren Tan)
Read in full from the downloaded `plugins-main` folder: `principle-sequence-verifiable-units`, `principle-prove-it-works`, and `docs/guide/06-verify-and-ship.md`.
- **Taken:**
  - state the finish condition up front, as checks the agent can run
  - small units that each end in a checkable state, checked before the next one
  - commits ordered so the sequence proves itself: the failing test first, then the fix
- **Dropped:** the Babysit and Shipping playbooks, and the overnight and swarm workflows

## Decided with the author, 2026-11-03
- The workflow is generic: plain skill files, a git hook, and no agent-specific tools.
- Prototype by default inside this workflow only, for screens, user flows and logic with more than two states; skipped with a reason for bug fixes, refactors, config and obvious plumbing. `design-docs` and `prototype` keep their own rules.
- Verification runs locally, with no CI. Answer keys and receipts stay in a git-ignored `.prove/` folder, and the keys are locked when the user approves them.
- The fresh checker falls back from a subagent to another agent's CLI to a new session; self-checked never counts as PROVED.
- Verdicts go in chat and screenshots in the PR's before-and-after table, following `ship` and `writing-pr`, instead of a verdict list in the PR body.

## Changed 2026-11-03, after a fresh-agent review
- The promise is now "no questions between the playback and ship's final checks"; ship's commit plan, interview and push stay, because they are checks.
- The verifier's method choice moves into the ask step, and the global hooks turn on first.
- `testing` joins the build step; a prototype is a structure key, never a pixel key; base-commit snapshots are taken before the lock; any commit inside ship means prove runs again.

## Changed 2026-11-03, after merge
- The quick start is now a table: each step, the skills it calls, what happens, what it hands on, and when it stops for the user. At the author's request.
- After a test run by a Cursor agent on a real feature (a blog tag filter), 2026-11-03: step 1 reads the real data as well as the code; step 2 creates the branch before the prototype, keeps the prototype running until the user picks, and checks its links first.
- 2026-11-03: step 7 says the user may skip the ship interview, matching `ship`. At the author's request.
- 2026-11-03, from the second half of the Cursor test run: step 4 runs every build step (not only screens) as a build, check, fix loop judged by a locked script or a fresh agent, at most 3 fixes, with no user in it; screen steps add screenshots and the pixel compare; each step names the Proof-table rows it finishes, scripts fail fast and a fresh agent judges only when they pass; step 3 saves the screenshot script, has a fresh agent open every snapshot, and proves every new check fails once before the lock. At the author's request.
- 2026-11-03, from the Cursor run's own report: step 6 makes all review fixes, then proves once on the new head.
