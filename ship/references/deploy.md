# Deploying

Read only when the user says the change is being deployed. The user runs the deploy. The agent writes the plan with them and, if asked, runs read-only checks afterwards.

## Before: the plan
Write it with the user, one decision at a time. Don't fill gaps with guesses.
- **Same build.** Build one artifact and promote that same one through each environment, with image and tool versions pinned rather than `latest`. Why: a rebuild per environment ships something nobody tested.
- **Undo.** How to turn it off: a flag (which one, and where it is switched), reverting the PR or MR (a revert commit, never a history rewrite), or redeploying the previous build. Name one, and how long it takes if the user knows. A database backup counts as an undo only after a restore of it has been run on a copy.
- **Migrations.** Does the old code still work against the new schema? If not, split the change: add first, switch the code, remove later. Run the down migration on a throwaway database to show it works. Say what happens to rows the new code has already written: kept, or cleaned up. Use the database skill's migration checks where installed.
- **Trigger.** What makes you undo it: the alert or SLO from the observability skill, or a bound on error rate and latency against today's baseline that the user agrees on. Don't invent a percentage.
- **Watch.** Who watches, where, and for how long after the deploy. Deploy when someone can watch it.
- **Budget.** If the service has an SLO, check the error budget left against the team's error budget policy. Don't invent a cut-off.

## Flags and stages, when the project has them
- Ship with the flag off, turn it on for the team, then widen in stages. The stage sizes and how long each one waits are the user's call. Ask.
- Move to the next stage only while the trigger stays quiet. Go back a stage, or turn the flag off, when it fires.
- Test both flag states. Give the flag an owner and a removal date, and remove the old path once it is fully on.

## After: checks (read-only, if asked)
Report each one with its number and where it came from:
- the health check responds, and it checks the real dependencies (a `/ping` that returns 200 while the database is down proves nothing). A gate with no data to read is a failed gate, not a pass.
- the error rate against the baseline, and no new error types
- the latency percentile against the baseline
- the main user flow, run by hand or with the automation skill
- logs from the new code are arriving, and the undo is still ready

Production is read-only for the agent. A fix or an undo is the user's action, or runs after their explicit yes.
