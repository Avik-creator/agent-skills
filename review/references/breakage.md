# What the change breaks outside the diff

Read when reviewing: the reviewer reads this on every review, and you read the last section after checking the proof (SKILL.md step 4). Listing callers is not the job, since a search finds them in seconds. The job is the breakage a search doesn't show. Findings here use the `compatibility` check, or `security` for a gate leak that exposes data or access, and need proof like any other ([proof.md](proof.md)).

## The fact it is safe because of
- A change that looks risky is usually safe because of one fact, such as "this only drops cache entries that have already expired". Find that fact before listing risks. If it holds, most of the risks are cleared at once.
- Prove the fact by running the real code: a script or test in the copy that calls the exact function and fails loudly if the fact is wrong. A cited line is weaker; your own explanation is not proof at all.
- If you can't prove it, the fact goes under Couldn't prove, with the command that would prove it.

## Where a search stops
- **Names that travel as data**: JSON fields an API returns, database columns, event and queue payloads, cache keys, config keys, URL routes, and anything another service, language or older client reads from the same bytes. A rename the compiler accepts still breaks these.
- **The library underneath**: read its source at the version the lockfile pins, including any local patch. Its docs may describe a different version.
- **When code runs**: teardown, retries, async ordering, startup and shutdown. A change can be right in the order it's written and wrong in the order it runs.
- **Further downstream**: follow a changed return value or side effect past its direct callers, to the code that finally uses it.

## Developer setup
A change that alters how people run or build the code locally is a finding:
- An environment variable renamed, added or newly required, or a secret read from a new place.
- A port, host or network setting moved.
- A new step needed before things work: a script to run, a service to start, or software installed by hand from outside the package manager.

Not a finding: a new optional way to run something, or a dependency the package manager installs as usual. Check that the setup files (`.env.example`, compose files, the README's setup steps) match what the code now reads.

## Feature gates
When the repo hides work behind a feature flag, an internal-only check or a plan check, look for gated code that now runs without its gate: a new route or export that skips the check, a default that flipped, a check moved after the work it guards, or a shared component that now renders gated content. These leaks are easy to miss, so trace every new entry point to its gate.

## Breakage the change means to make
- Don't report breakage the intent asks for, such as removing a flag, a feature or a safeguard, when the change is limited to that. Why: it costs the author time and teaches them to skip your findings.
- Do report it when the impact reaches further than the intent suggests the author knows, when the change is far larger than its description, or when it looks deliberately harmful.

## Finish the research
When the answer depends on code you can read, read it. Never report "the client sends X, which is fine if the server rejects it" when the server is in the repo. Either prove it breaks, or clear it.

## Existing PR comments
You read these, not the reviewer, and only after step 4. Why: a reviewer who reads them first follows them instead of looking for itself.
- On a PR, read its discussion (`gh pr view <n> --comments`, `glab mr view <n> --comments`), including bot reviewers.
- A point the reviewer missed: prove it as any finding, then add it with "(also raised by <author>)". One that doesn't hold up goes under Disputed.
- A point the reviewer also found: keep one finding, and credit the author.
