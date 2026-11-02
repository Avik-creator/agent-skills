# Sources

Ideas were adapted and re-checked, not copied. Pulled 2026-11-02 from local downloads of addyosmani/agent-skills (`agent-skills-main`) and wshobson/agents (`agents-main`); no commit SHA was recorded for those zips.

| Upstream | License | Taken |
|---|---|---|
| addyosmani/agent-skills `references/definition-of-done.md` | MIT, Copyright (c) 2025 Addy Osmani | The done gate: run at runtime, a test that fails without the change, no regressions, scope kept, migrations, config and flags accounted for, backward compatibility stated, docs, security and telemetry where relevant, the human approves before merge |
| addyosmani/agent-skills `skills/constraint-driven-development` (SKILL.md, `references/floor-guard.md`) | MIT, same | "Guard the bar": the moves an agent makes to get to green (suppressions, skips, deleted tests, removed assertions, stubs, an empty catch, a lowered threshold). `scripts/bar-guard` is a new, smaller shell version: it reads untracked files too, and its exit 2 never reads as clean |
| addyosmani/agent-skills `skills/git-workflow-and-versioning` | MIT, same | One logical change per commit; refactors apart from behaviour; a message body that says why; the staged-diff secrets check; semver judged by what consumers observe; the tag as the source of truth; a changelog written for consumers in the same change |
| addyosmani/agent-skills `skills/shipping-and-launch` | MIT, same | `deploy.md`: an undo plan before the deploy, flag off first, staged widening, both flag states tested, a flag owner and removal date, checks after the deploy, the error budget checked against the team's policy |
| addyosmani/agent-skills `skills/interview-me` | MIT, same | One question at a time, and "I don't know" as a valid answer. The direction is reversed here: the agent examines the user's understanding instead of drawing out their intent |
| wshobson/agents `plugins/ship-mate` (all 14 files) | MIT, Pranay Yadav | A fix that changes a schema, API, auth or scope is asked about first; a plan step that can't be followed is flagged, not skipped |
| wshobson/agents `plugins/git-pr-workflows/commands/git-workflow.md` (rules, push and PR phases, rollback read in full; the rest by headings) | MIT | Show the exact git and PR commands and the description, then run them after a yes; a revert commit rather than a rewrite |
| wshobson/agents `plugins/block-no-verify` | MIT | Never `--no-verify`, as a rule. The hook config itself is not copied |
| Author's library rules | n/a | The examiner interview, graded against the code; it teaches and never blocks; a pre-push record checked on every push, with a short re-interview after new commits and a patch-id match after a rebase; `SHIP_SKIP=1` as the user's escape; the user merges, deploys and tags; no AI attribution; never push to the default branch; cleanup shown before deletion |

## What was dropped, and why

- shipping-and-launch's rollout thresholds (error rate "within 10%", "> 2x baseline", p95 "+20%"/"+50%", client errors "0.1% of sessions", business metrics "5%"), the "24-48 hour" and "1 week" windows, the "5% → 25% → 50% → 100%" stages, the rollback times "< 1 / 5 / 15 minutes", and the "> 20%" error-budget gate. None is sourced. The user sets the stages and the trigger.
- shipping-and-launch's pre-launch checklists (security, performance, accessibility, infrastructure) and its React error-boundary code. Security, telemetry and performance belong to their own skills; the done gate points at them.
- "Clean up flags within 2 weeks" and "It's Friday afternoon". The first is invented; the second is replaced by "deploy when someone can watch it".
- git-workflow-and-versioning's trunk-based "push to main" option, which conflicts with never pushing to the default branch; its ~100 / ~300 / ~1000-line PR sizes (invented; `review` keeps its own split suggestion); "DORA research consistently shows" (not re-checked); `git reset --hard HEAD` as a save-point move, which destroys work; and the lint-staged/husky setup.
- constraint-driven-development's CONSTRAINTS.md interview, the scanner installs, the npm scripts, `quality-gate.mjs`, and the default numbers (80% changed-line coverage, 60% mutation score, 90-day exception lifetime, 0.5% ratchet tolerance), which have no source. Scanners are the security skill's.
- interview-me's hypothesis with a confidence number, the guess attached to each question, and the statement-of-intent restate. They are for drawing out intent; in an examiner interview they would give the answers away. Its intent-drawing ideas fit design-docs `interview.md`, offered separately.
- ship-mate's seven-agent pipeline, state.json, story files, AGENTS.md generation, the fixed FRONTEND/BACKEND split, the 🟡 findings fixed without asking (`review` owns findings), the MERN notes, and Playwright page objects (end-to-end tests are the testing skills' area).
- git-workflow.md's eight-subagent phases and `.git-workflow/` files. A single show-then-run step covers the same checkpoints.
- block-no-verify's PreToolUse hook. It is a settings change; the rule and the pre-push hook cover the need.
- before-you-build. design-docs `ideas.md` already covers demand, assumptions and the pre-mortem.
- ci-cd-and-automation, deprecation-and-migration, pr-enhance, deployment-pipeline-design: outline only. They are CI templates, migration patterns (the database skill) or PR templates the description rule already covers.

## Verified external facts

Checked on 2026-11-02.

| Fact | Where checked |
|---|---|
| MAJOR for "incompatible API changes", MINOR for "functionality in a backward compatible manner", PATCH for "backward compatible bug fixes"; "Major version zero (0.y.z) is for initial development. Anything MAY change at any time." | https://semver.org/ |
| Change types Added, Changed, Deprecated, Removed, Fixed, Security; "The latest version comes first"; keep an `Unreleased` section at the top | https://keepachangelog.com/en/1.1.0/ |
| `gh pr create` flags `--base`, `--title`, `--body-file`, `--draft` (gh 2.96.0) | `gh pr create --help` on this machine |
| `glab mr create` flags `--target-branch`, `--title`, `--description`, `--draft`, `--yes`; `--fill` "pushes the branch"; `--push` pushes committed changes (glab 1.107.0) | `glab mr create --help` on this machine |
| A pre-push hook reads `<local ref> <local sha> <remote ref> <remote sha>` lines on stdin; a deleted ref has an all-zero sha | Behaviour exercised in the tests below (git 2.54.0) |

## Tested

In throwaway repos with a local bare remote, on git 2.54.0:
- `pre-push`: no record blocks; a matching record passes; a new commit blocks and is listed; `SHIP_SKIP=1` passes; a rebase onto a new main with the same change passes and moves the record; an amend with different content blocks; tags and branch deletes are not checked; a branch name with slashes works.
- `install-hook`: keeps an existing hook, which still receives the same stdin; is safe to repeat; works from a linked worktree; exits 3 with no change when `core.hooksPath` is set.
- The hook-manager snippet: does nothing for a teammate without the local check, blocks without a record, and passes with one.
- `bar-guard`: clean is 0; it catches `# noqa`, "Not implemented", `test.skip`, `t.Skip`, a removed `expect`/`t.Fatal`, a deleted test file, an empty catch and `TODO` in an untracked file whose name has a space, and doesn't flag `TODOS`; a bad base is 2. A bug found in testing (`awk -v` stripped the backslashes, and the awk error reported "clean") was fixed: patterns now go through the environment, and an awk failure exits 2.

Not run: the real `gh pr create` and `glab mr create` calls, and the interview itself (no with/without test yet).

## Refresh

1. Re-read semver.org and keepachangelog.com if `release.md` changes.
2. Re-check `gh pr create --help` and `glab mr create --help` after a major CLI upgrade, especially whether any flag starts pushing.
3. Re-run the hook tests above after changing `scripts/`.

## Added 2026-11-02 (source sweep)
- From addyosmani/agent-skills `commands/build.toml` and qa-skills-main `docs/QA-RUNNER.md`: stage only the commit's files, a hedged answer is not a yes, a pass only on retry or a stale result is not a pass.
- From wshobson/agents `cicd-automation/deployment-pipeline-design` and `database-design/database-admin.md`: build once and promote the same artifact, pin versions, a health check that tests real dependencies, a gate with no data fails, a backup is an undo only after a restore. Their percentage cut-offs were dropped.

## Added 2026-11-02 (engineering-method and writing-pr)
- From `engineering-method` §6, in done.md: search for the old name again after a rename, and re-read the whole diff once.
- Step 5 now writes the description with `writing-pr`. The old "how it was verified" line was removed, because the author's writing-pr rule forbids it in the body.
