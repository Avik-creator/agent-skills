# System design and architecture reviews

Read when writing a system design, reviewing an architecture, or proposing a scale or performance improvement.

Write as a senior engineer: recommend the smallest design that fits the evidence, name what you did not choose and why, and never reach for a pattern the evidence doesn't justify. Queues, caches, workers, sharding and microservices each need a named, concrete reason.

## Pick the mode

Ask which mode when the request doesn't say.

**As it is.** Describe the system that exists today.
- Read the repo: entry points, callers, routes, data stores, integrations, background jobs and deploy config.
- Every component cites the file it comes from. A part you couldn't find goes under Open questions.

**A proposal.** Design something new.
- First ask for the constraints that would change the design: expected scale, latency target, budget, team size, what already exists, and what must not change. Give your recommended answer with each.
- Sketch at least two structurally different designs before choosing: different owners, boundaries or data flow, not one shape with a tweak. Why: the first shape that works is rarely the only one, and a second one shows what the first costs.
- Recommend one design. Name the alternatives you didn't choose, one line each, with the reason.
- Never invent users, scale or a deadline.

**An improvement.** Make an existing system faster, cheaper or sturdier.
- Read the current design from the code first.
- Then gather real usage: traffic, growth, hot paths, read/write ratio, latency, error rate and cost. Use what the repo already exposes: logging and metrics code, dashboard config, tracing setup. Ask the user for any number the repo doesn't show.
- Never estimate a number that neither the repo nor the user gave you. It goes under Open questions.

## Facts and cost

- Search the web before relying on an external fact: pricing, quotas, service limits, version behaviour, published benchmarks. Say what you found, and the date. Why: stale prices and limits produce a design that costs the wrong amount.
- Treat cost as a requirement, not a footnote. Put real numbers against each option: monthly infrastructure, per request, per GB, licences. An option that blows the budget is not an option.
- Name the constraint behind each choice, so a change in budget or scale points straight at what to revisit.

## Design in layers

- V1 is the simplest design that meets today's requirement. Add a layer only for a failure or bottleneck that evidence shows.
- No performance claim without a measurement. Give the number, its unit, and how it was measured. If nothing was measured, say "not measured", and treat the optimisation as unjustified.

## Screen the shape

Check each candidate design for these, and revise or reject a shape that has one. Assume the next person to change the code sees only the file they opened and copies the nearest example; prefer the design where a change that looks right in one file is right everywhere.

- **Shallow module**: a large interface that hides little. Callers coordinate several calls to finish one operation, or options expose internal stages. Prefer a small interface over substantial behaviour.
- **Leaking details**: a representation, policy or wire format that several modules depend on, so changing it means coordinated edits. Parse outside data into the system's own types at the boundary, and keep storage and protocol details private.
- **Ordered by steps**: modules named for when they run (load, validate, transform, save) that each repeat the same rules. Group code by the knowledge it owns instead.
- **Pass-through layer**: a method or layer that forwards the same arguments to another with the same shape. Remove it, unless it adds policy or a real abstraction.
- **Split ownership**: more than one module writes the same state or keeps its own copy. Give each piece of state one owner that others read from or ask.
- **Two ways to do one task**: every way keeps gaining callers. Keep one, and move callers off the others in the same change.
- **Importable internals**: outside code can reach a module's internals, so they become part of its interface. Make an outside import fail the build.
- **Hand-synced lists**: two places list the same items. Derive one from the other, or make the build fail when they disagree.

When two shapes both pass, prefer the one that hides more behind a smaller public interface.

## The document

Use only the sections that apply:

- the job the system does
- the components, and what each one owns
- the request path, step by step (see flows-and-diagrams.md)
- where data lives, and who can change it
- failure: what is lost, what retries, and what the user sees
- what sits outside the system (third parties, other teams)
- trade-offs, and what each choice enables or rules out later
- cost per option
- open questions

## Failure modes

Walk this list, and for each item say "applies, because..." or "doesn't apply, because...":

- invalid input
- missing data
- duplicate requests and retries
- partial failure
- timeouts
- concurrency and race conditions
- stale data
- permissions
- network or dependency failure
- deployment and rollback
- observability (how you would know it broke)

## For an improvement, also add

- the evidence you read
- each proposed change, tied to the evidence that justifies it
- a priority for each change:
  - P0: a risk the evidence shows is already happening
  - P1: a real but smaller gap
  - P2: polish

A change with no evidence behind it is not proposed.

## Scope

- A security need gets one line: flag it for a security review (the `security` skill's design review, where installed).
- Database internals follow the `database` skill where it is installed.
- Building the change happens only after the user approves the design.
