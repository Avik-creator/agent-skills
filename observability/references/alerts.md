# Alerts

Read when writing or changing an alert. Creating the alert object, the dashboard, or a saved view in a real backend is that backend's skill. Hand it over after a yes, with the rules below as the brief. Why: a page to a shared channel is outward-facing.

## Symptom, two severities

Page on what a user would feel: error rate, a latency percentile, how old the oldest queued job is. A cause (CPU, a restart, disk) goes on a dashboard. Why: a cause page fires when users are fine, and stays quiet for a failure you did not predict.

Two severities:

- **Page.** Users are affected now. Someone acts.
- **Ticket.** Degraded. Someone acts on this week's queue.

A third severity becomes a channel people mute.

## The threshold

The threshold and how long it must stay true come from an SLO or from this service's history. Write the source next to the alert ("error budget for the checkout SLO", "p95 over the last 30 days plus the headroom we agreed"). A number you picked because it looks serious is a guess. Delete it and ask. For an SLO-based threshold, use the burn-rate windows in [slo.md](slo.md).

Every alert is actionable. If the response is to wait for it to clear, delete the alert.

## Runbook

Link a runbook. The shape is the one in design-docs `references/documentation.md`: the symptom someone sees, the checks in order, the fix, how to confirm it worked, and who to call. Do not invent a second format. Write each check as a command to run or a place to look.

## Test-fire

Fire the new alert once, against the channel it will use, and open the runbook link from the message. A threshold you lowered for the test goes back. Why: an alert that never fired is a wish.

Do this in the backend's own skill when that skill is what creates the object. This skill stops at the rule and the handoff.
