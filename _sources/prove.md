# prove: sources

Written 2026-11-03. Ideas adapted, not copied. Every file was rewritten in this library's style and size rules.

## Ray Amjad: ray-amjad/skills (MIT, Copyright (c) 2026 Ray Amjad)
Read in full from the downloaded `skills-main` folder: `create-prove-it`, `maintain-prove-it`, `audit-prove-it`, `task-lifecycle/references/verify-round.md`, `create-prove-it/references/methods.md`, and the README.
- **Taken:**
  - surface, answer key and comparison as the three parts of a check, and the five-rung answer-key ladder
  - checking only what the change touches: new flows against the spec or parity, existing flows against the app before the change, a smoke test for new-files-only changes
  - the verdicts PROVED, BROKEN and BLOCKED, plus UNPROVEN when there is no key; never a weaker check in place of one that can't run
  - the method table (browser, HTTP, CLI, stored state, logs, side-effect capture, snapshot, parity) with catches, misses and cost, and the brainstorm table the user picks from
  - a verifier and a checker kept apart: route on the checker's verdict; the checker never sees the builder's reasoning
  - outcomes after a verdict: a pass with defects, a capped fix loop, no fix loop for a missing credential, unreachable states reported apart from failures
  - receipts per run; drift versus product bug when the app and the verifier disagree; never rewriting a key to match the app
  - the audit finding that checks against a single record miss most planted bugs
- **Changed:**
  - everything lives in a local, git-ignored `.prove/` folder in the main checkout, not a committed project-local skill, at the author's request
  - answer keys are locked with fingerprints (`scripts/keys`) and a pre-push hook (`scripts/pre-push`) blocks unproved pushes. Both are new, and work with any agent
  - the fresh checker falls back from a subagent to another agent's CLI to a new session; "self-checked" never counts as PROVED
- **Dropped:**
  - the temporary migration skill and its end step; a migration is one row in answer-keys.md
  - `maintain-prove-it` and `audit-prove-it` as separate skills; their core rules are in setup.md
  - `task-lifecycle` (it overlaps `review` and `ship`, and it merges)
  - the course links and the beta notes

## pstack, in cursor/plugins (MIT, Copyright (c) 2026 Lauren Tan)
Read in full from the downloaded `plugins-main` folder: `create-verification-skill`, `principle-prove-it-works`, `poteto-mode/playbooks/visual-parity.md`, and `docs/guide/06-verify-and-ship.md`.
- **Taken:**
  - ask the repo, not the user
  - the Launch, Doctor, Drive, Evidence and Cleanup sections, ready meaning usable, stable handles, and cleanup that kills only what it started and never deletes the evidence
  - the feature map with "done looks like" and Gotchas; never weakening a journey to pass
  - proving the generated verifier once before handing it over
  - "a check you have never seen fail proves nothing"; don't trust a dry-run by its name
  - the anti-shortcut rules from visual parity: no harness edits, no baseline tampering, no restructuring to pass a diff; 0% for parity work
- **Dropped:** the `.cursor/skills/` location, the formal-verification skills, and the Babysit and Shipping playbooks

## Facts checked
- `codex exec` and `claude -p` checked with `--help` on 2026-11-03. Cursor's CLI was not installed, so it is not named.
- `scripts/keys`, `scripts/pre-push` and `scripts/install-hook` tested in throwaway repos on 2026-11-03: lock and check, edited and added keys, subfolders and linked worktrees, each receipt result, self-checked runs, skipped pushes, branch deletes and tags, a missing `.prove/`, and real pushes chained with the `ship` hook in both install orders.

## Changed 2026-11-03, after review and at the author's request
- **Global hooks:** the per-repo installer is gone. The library's `_hooks/` folder is set as git's global `core.hooksPath` by `_hooks/install`, which `prove`, `ship` and `deliver` run without asking, at the author's request ("I don't want any check to be missed"). Its `pre-push` runs the ship and prove checks and then the repo's own hook; every other hook name passes through to the repo's own hook.
- **Fixes from a fresh-agent review, each tested in throwaway repos:**
  - the gate keys on the remote ref, so `git push origin HEAD` and pushing a raw commit are checked (ship's hook had the same gap)
  - one lock per task (`keys lock <task>`); a deleted keys folder, an empty task and a bad task name are refused
  - the receipt names its task; `checker` is an allow-list (subagent, script, session, cli:<name>); an unagreed `not covered` line blocks PROVED; Windows line endings are read
  - the main checkout is found with `--show-toplevel`, or the first `worktree list` entry from a linked worktree, so a separate git folder works
  - `scripts/init` creates `.prove/` and adds it to the exclude file once
  - the pass-through hook is silent when git runs it before a repo exists (during `git init` and `git clone`)
- **Text:** a `script` checker counts only for checks locked with the keys; a prototype is a structure key, never a pixel key; snapshots are taken before the lock; the smoke test is keyed to the base commit; `verifier.md` changes only through the drift rule, reported to the user.
- 2026-11-03, after merge: the missing-`.prove/` check moved inside the per-ref loop. Before, a repo where prove had never run blocked every push, branch deletes and tags included. Found by the live check after PR #9.
- 2026-11-03: a NO-SURFACE receipt now needs a fresh-agent checker (subagent, session or cli:<name>). Before, a receipt with no checker line passed, so the builder could skip proof by saying nothing runs.
- 2026-11-03, found in a test run by a Cursor agent on a real feature (a blog tag filter): answer-keys.md says a key from the code under test is not a key, a list check is equality, and new UI that shifts a screen is checked by element screenshots of the unchanged parts at 0%; setup.md prefers the production build for checks.
- 2026-11-03, from the second half of the Cursor test run: answer-keys.md says open every snapshot before the lock, freeze animations after load and read back opacity, and a pixel pass needs a fresh agent's eyes (two blank images match at 0%); methods.md says compare decoded pixels after checking dimensions, never file bytes. checker.md and SKILL.md say a pixel diff never counts as the `script` checker on its own.
- 2026-11-03, from the Cursor run's own report: checker.md says evidence file names must differ in more than letter case (flow-go.json and flow-GO.json collided on macOS).
