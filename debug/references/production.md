# Looking at a live system

Read before looking at production or any shared environment. Here, debug only reads. Why: a restart, rollback or cleanup changes the evidence and can turn one outage into two.

## Read-only means
- No deploy, restart, rollback, scale change, flag flip, or config change.
- No writes, `UPDATE`, `DELETE`, killed sessions, cleanup jobs, cache flushes, or `VACUUM`.
- No changes to DNS, network, firewall, or secrets.
- Queries are reads with a time limit and a row limit. Prefer a replica where one exists.
- Production data stays where it is. A copy for a reproduction needs the user's yes and has personal data removed.

If the only next step is one of these, stop and propose it (below).

## Order of signals
1. **Metrics**: is it broken, since when, and for whom? Look at error rate, latency and saturation by version, region and endpoint.
2. **Traces**: where does the time or the error sit, and is it this service or one it calls?
3. **Logs**: why, for the requests the traces point at.

Build a timeline: first seen, then every deploy, config change, flag change, migration and traffic shift on that line. Ask whether the error starts in an upstream service.

## Claims
- Label each claim observed, inferred, or unknown.
- One probe is a data point, not a cause. "The health check passed once" doesn't show the service is healthy.

## Timing a request with curl
`curl -o /dev/null -s -w '%{time_namelookup} %{time_connect} %{time_appconnect} %{time_starttransfer} %{time_total}\n' <url>`

Each value is measured from the start of the request, so the values add up. Subtract neighbours to get each phase: DNS, then TCP connect, then TLS, then server work plus the first byte, then the transfer. `time_starttransfer` includes the time the server spent computing the answer. From the curl manual, checked against curl 8.7.1.

## Proposing a mitigation
A rollback, flag flip, scale-up or traffic shift is a proposal for the user to approve, never a step you take. Write it as:
```
Action:        <the exact command or change>
Expected:      <what it should change, and the signal that shows it worked>
Undo:          <how to reverse it>
Leaves broken: <what it does not fix>
```
The code fix still goes to `coding`, starting with a reproduction test that fails.

## Handing off
When stopping before the cause is found, leave the symptom, the timeline so far, what was checked and ruled out, what is unknown, and the next check. A full postmortem document is `design-docs`, and only when the user asks for one.
