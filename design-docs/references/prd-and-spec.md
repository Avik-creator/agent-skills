# PRDs and engineering specs

Read when writing a PRD (what to build, and why) or an engineering spec (how the system will behave).

## Which one

- **PRD:** says what will be built and why. It does not choose the implementation. The readers are product, design, engineering and stakeholders.
- **Engineering spec:** says how the system will behave: interfaces, data, failure, trade-offs, rollout and verification. The readers are the engineers who build and review it.
- A system design or architecture review goes to system-design.md instead.

## PRD sections

Use only the sections that apply:

- Summary
- Problem and evidence
- Users
- Goals
- Non-goals
- User flow
- Requirements with acceptance criteria
- Success measures
- Constraints and dependencies
- Risks
- Proof
- Open questions

## Engineering spec sections

Use only the sections that apply:

- Summary
- Current behaviour
- Proposed behaviour
- Flow and interfaces
- Data and failure handling
- Trade-offs
- Compatibility
- Rollout and rollback
- Proof
- Open questions

## Rules

- **Goals:** a goal is a result, not an activity. "New users finish setup in under 5 minutes", not "Improve onboarding".
- **Requirements:** each one states behaviour you can observe, and how to tell pass from fail. Add the unhappy path wherever bad input, an empty state, a failure or a permission matters.
- **Non-goals:** a non-goal excludes something a reasonable reader would assume is included.
- **Success measures:** each one says what changes, from what to what if known, and how it will be checked.
- **Trade-offs:** for each choice, name the option not taken and why, in one line.
- **Proof:** every goal and requirement gets a row in the Proof section, as proof.md says.
- **Rollout:** include a way back (a flag, a revert, a migration down) and what to watch while rolling out.
- **Open questions:** name what each one blocks. Name an owner only when one is known.
- **The end:** finish with the decision still needed before implementation.
- **Placeholders:** no TBD anywhere except Open questions.

## Example requirement

Weak: "Users can reset their password."

Good:

- A signed-out user who enters a registered email gets a reset link within 1 minute.
- The link works once, for 30 minutes.
- An unregistered email shows the same message, so no one can find out which emails have accounts.
- Pass: all three checked in staging.
