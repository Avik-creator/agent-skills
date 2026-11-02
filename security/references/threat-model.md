# Threat model

Read when starting a feature that crosses a trust boundary, when placing a finding on the risk grid, or when writing the security section of a design doc.

## Boundaries and assets

A trust boundary is a place untrusted data enters code you run. Count a request, a form field, an upload, a webhook, a third-party response, a queue message, model output, another process's command line or environment, a filename on a shared volume, and a path inside a job payload. A value is not trusted because the operating system handed it to you.

Name the assets someone would steal or break: credentials, personal data, payment data, admin actions, money movement.

## STRIDE per interaction

Start from the letters in the row, and add one when it fits this system. Spoofing, tampering, repudiation, information disclosure, denial of service, elevation of privilege.

| From | To | Letters | Ask |
|---|---|---|---|
| External | Process | S, T, R, I, D, E | Can the caller pretend to be someone else, alter the payload, deny sending it, read it on the way, flood the process, or come out with more rights? |
| Process | Datastore | T, R, I, D | Can the process alter stored data, deny a write, read data it should not, or fill the store? |
| Process | Outbound call | T, I, D | Can the caller choose where the server connects (SSRF), can the reply be altered or read on the way, can a slow or huge reply stall the process? |
| Process | Process | S, T, I, D, E | Can one process pretend to be another, alter what it sends, leak data across, overwhelm the other, or gain the other's rights? |

The rows are a starting point, not a rule. A letter left out still needs a look when the system gives it a reason, such as repudiation between two services that act for different users.

## Abuse cases

For each feature, write how you would misuse it, two to five cases. Each case names the caller, the input, and the bad outcome. Each case becomes a test that fails before the fix exists.

A shape check is not an abuse case. "The path must be absolute" does not show that a delete stays inside the allowed root.

## Impact × likelihood

Use three levels for each: low, medium, high. Say the cell in words ("high impact, low likelihood"). Do not turn the cell into a score.

Propose a fix first for a finding that is high on both. Anything you defer, including a high finding you cannot reach or cannot patch yet, gets a reason and a review date. The user decides the order.

## Control types

Tag every control you keep:

- **Preventive** stops the action. An ownership check, a parameterized query.
- **Detective** shows that it happened. An audit event for a denied login.
- **Corrective** limits the damage after. Rotating a secret that leaked, disabling a session.

If a threat has a preventive control and nothing that would show or limit a failure of that control, write the residual risk.

## Residual risk

One or two sentences: what you did not close, who accepts it, and when it is reviewed. "None" is allowed when you can point at the test that fails if the abuse works.

## Design-review section

Discuss the section with the user, then add it to the design doc through the design-docs skill. Omit a heading that would not change a decision.

```markdown
## Security
Boundaries: <who writes each value that crosses in>
Assets: <what is worth stealing or breaking>
STRIDE: <interaction, letters, control>
Abuse cases: <case, and the test that fails first>
Controls: <name, preventive | detective | corrective>
Residual risk: <what remains, who accepts it, review date>
Open: <questions for the owner, including which regulations may apply>
```
