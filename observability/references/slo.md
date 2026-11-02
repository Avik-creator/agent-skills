# SLOs and burn-rate alerts

Read when an alert's threshold should come from an SLO, or when the user asks for an SLI, SLO or error budget. The target is the user's decision; this file gives the method and the arithmetic.

## The SLI
An SLI is a ratio: good events divided by total events, so it runs from 0 to 100%. Pick five or fewer per service. Types, as the SRE Workbook names them:

| Service | SLI | Good event |
|---|---|---|
| Request-driven | Availability | the request got a successful response |
| Request-driven | Latency | the request was faster than a threshold |
| Request-driven | Quality | the response was served undegraded |
| Pipeline | Freshness | the data was updated more recently than a threshold |
| Pipeline | Correctness | a record going in produced the correct value coming out |
| Pipeline | Coverage | a batch job processed above a target amount of data |
| Storage | Durability | a written record can be read back |

- Measure as close to the user as you can: the load balancer or the client, not the application server. Why: the server can't see requests that never reached it.
- Leave out your own health checks and probes. Why: they succeed while users fail, and they inflate the ratio.
- Write down what counts as good, and the threshold, with the SLI. "Latency" without the threshold is not an SLI.

## The SLO and the error budget
- The SLO is a target for the SLI over a stated window, such as 30 days. Ask the user for both. Don't pick them.
- The error budget is 100% minus the SLO. Show it in time or events so the user can judge it: 99.9% over 30 days is 0.1% × 43,200 min = 43.2 minutes.
- What happens when the budget runs out is an **error budget policy**. The team writes it and the people who own the product, the code and on-call agree to it. Offer to draft it with the user; don't invent its thresholds. It says what changes when the budget is spent, who decides, and who settles a disagreement about the numbers.

## Burn-rate alerts
Burn rate is how fast the service spends its budget, relative to the SLO. A burn rate of 1 spends exactly the budget over the SLO window.

Budget spent = burn rate × alert window ÷ SLO window. Alert on budget spent, using two windows: the long window says the burn is real, and the short window (1/12 of the long one) says it is still happening, so the alert clears soon after the fix.

The SRE Workbook's starting point, for a 99.9% SLO over 30 days:

| Severity | Long window | Short window | Burn rate | Budget spent |
|---|---|---|---|---|
| Page | 1 hour | 5 minutes | 14.4 | 2% |
| Page | 6 hours | 30 minutes | 6 | 5% |
| Ticket | 3 days | 6 hours | 1 | 10% |

These map onto the two severities in [alerts.md](alerts.md). For another SLO or window, keep the budget-spent column and recompute the burn rate with the formula above; show the user the working.

## When the numbers don't work
- **Low traffic.** Few requests make the ratio jump, and the alert flaps. The Workbook's options: generate artificial traffic, combine small services into one for monitoring, or change the product so a single failure counts for less. Ask which fits.
- **A low target** (such as 90%). A full outage may spend less than the page threshold in an hour, so the page never fires. Say so, and check the table against the target before using it.
- **A very high target** (such as 99.999%). A full outage spends a month's budget in under a minute, faster than anyone can respond. Say so; the target may be the thing to change.
