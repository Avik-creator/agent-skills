# Documentation for something that exists

Read when documenting existing behaviour: a flow, an API reference, a runbook, an onboarding guide, a README, a changelog, a user guide, or another document the user names.

## Read first

Before writing, read the code:

- entry points, callers and routes
- data stores and integrations
- error paths, feature flags and tests

Document what the code does, not what its comments or old docs claim. Why: comments and old docs drift, and the code is what runs.

## What each type contains

- **Flow:** its purpose, who takes part, what starts it, the steps, inputs and outputs, how data moves, what can fail, and how to recover. Use the format in flows-and-diagrams.md.
- **API reference:** for each endpoint:
  - the method and path
  - who may call it
  - the inputs, with types and which are required
  - the output
  - each error, and when it happens
  - one example request and response
- **Runbook:** the symptom someone sees, the checks in order, the fix, how to confirm it worked, and who to call when it doesn't.
  - Write each check as a command to run, or a place to look.
  - Say what must be true before a step (access, a tool, a state), and what to do when its command fails.
- **Guide or README:** what it is, who it is for, how to set it up, how to use it, and how to tell it worked.
- **Changelog:** what changed for the reader, grouped as added, changed, fixed, and removed. Leave out internal refactors the reader never sees.

## Always include

- the dependencies
- the access rules the code shows (who may do what)
- the monitoring (how someone knows it is broken)
- where each fact came from, marked confirmed or assumed. A number or quote must be in the source it cites, and goes stale when that code changes.

When editing a draft, keep its qualifiers ("usually", "for admins only", "since v2"). A sentence that loses one is a new, wrong claim.

## Example runbook entry

**Symptom:** checkout pages show "Payment unavailable".

1. **Is the payment provider down?** Check their status page (link).
2. **Are our calls to the provider failing?** Search the logs for `payment_client error` in the last 15 minutes.
3. **Has the API key expired?** The logs show `401` from the provider.
   - Fix: rotate the key in the secrets manager, then restart the checkout service.
   - Confirm: place a test order with a test card.
