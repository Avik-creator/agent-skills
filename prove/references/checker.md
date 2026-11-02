# The fresh checker

Read before every verdict. The agent that built the change shares its blind spots, so it never gives the verdict. Scripts and a fresh agent do.

## Who checks, in this order
Use the first one available, and write which one in the receipt's `checker` line.
1. **`script`:** every flow in scope is fully covered by scripted checks with a pass signal. No judgment is needed.
2. **`subagent`:** a new subagent with fresh context, where the agent supports them.
3. **`cli:<name>`:** another agent's CLI run headless with only the brief, for example `codex exec` or `claude -p`, or any other agent's non-interactive mode. A different model from the builder catches more.
4. **`session`:** write the brief to `runs/<run>/brief.md` and ask the user to paste it into a new session, then read its answer.
5. **`self`:** none of the above was possible. Every flow that needed judgment is UNPROVEN.

## The brief
The checker gets only:
- the flows in scope, each with its answer key and pass signal
- the evidence paths: screenshots, request and response files, command output, the stored-state query and its result, logs
- the rules: route on what the evidence shows; a step without evidence fails; a prose claim is not evidence

It never gets the builder's reasoning, the diff explanation or the implementation notes. Why: the checker must judge what happened, not what was meant.

## What the checker returns
For each flow: the verdict (PROVED, BROKEN, BLOCKED or UNPROVEN), the evidence line it relied on, and any defect seen beyond the pass signal.
- **PROVED with defects** is still a pass. Report each defect to the user; fix it only when they ask, or when it is a few lines in a file the change already touches.
- **BROKEN:** one fixer works from the evidence, then the flow is proved again. At most 3 rounds, then stop and report.
- **BLOCKED:** no fix loop, because no code change fixes a missing credential. Say plainly the flow is unverified.
- **Unreachable state** (the precondition is a state the app never creates): report it as unreachable, not broken, and ask whether that code path should exist.

## Evidence that counts
- **A screen:** numbered screenshots or a recording of the flow, at each width the plan names.
- **An API or job:** the real request, response and a stored-state query in one text file (`... 2>&1 | tee runs/<run>/flow-<n>.txt`).
- **Always:** a read-back of the data that should have changed, not only a success message.
