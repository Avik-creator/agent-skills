# Checks

Read when reviewing: the reviewer reads this, and you read it when picking checks for two reviewers. These are questions to ask of the diff. A yes becomes a finding only with proof ([proof.md](proof.md)).

Skip anything a linter, formatter or type checker already enforces. Run the tool and report its result instead. Why: a review that repeats the linter buries the findings only a person can make.

## Correctness
- Does each changed path do what the intent bullets say, including errors, empty input, and boundary values?
- What happens on a retry, a duplicate request, a timeout, or a partial failure halfway through?
- Can two requests or workers interleave here (a race, a check-then-act gap)?
- Is a fallback or default hiding a case the code should refuse?
- Off-by-one, wrong comparison, wrong unit (seconds against milliseconds), null or missing data.
- What the diff doesn't show: a file, connection, transaction or subscription it opens and doesn't close on every path, including errors; a method or import it calls that doesn't exist in the version the lockfile installs.

## Security
These find gaps in a diff; building the controls and threat-modelling them is the `security` skill, where installed.
- **Trust boundaries**: where does data come from outside (requests, uploads, webhooks, third-party APIs, model output, files other processes write)? Is it validated there, with an allowlist?
- **Access**: is every new endpoint or action checked for who is calling it? Is every lookup scoped to the caller's own records and tenant? Why: a missing ownership check lets one user read another's data by changing an id.
- **Injection**: queries parameterized; no input built into shell commands, `eval`, HTML or file paths.
- **Outbound**: URLs fetched by the server or used for redirects are allowlisted. Why: otherwise the server can be pointed at internal addresses.
- **Paths**: deletes, moves and overwrites named by data resolve symlinks first, and stay inside an allowed root.
- **Secrets and leaks**: no secrets in the diff; no tokens, passwords or personal data in logs, errors or API responses; errors sent to clients carry no stack traces or queries.
- **Model output**: treated as untrusted input; permissions enforced in code, not in a prompt.

## Performance
- A query inside a loop (N+1), a list with no limit or pagination, a loop of single calls where a bulk call exists.
- Heavy or blocking work in a request handler or on a UI's main thread.
- A cache: is its key missing an input the response varies on (tenant, user, permissions, locale)? Is it unbounded? Why: a key without the user serves one user's data to another, and an unbounded cache is a memory leak.
- A connection pool or client created per request.
- Front end: a heavy import on the initial load, re-rendering everything on each change, or reading and writing layout in a loop.

A performance finding needs a measurement. Without one it goes to Couldn't prove.

## Tests
For the edge cases and error kinds to look for, use the testing skill's `references/error-hunt.md`, where installed.
- Do tests exist for the changed behaviour, and would they fail if it broke? Answer with a mutation, not by reading.
- Do they test what a user or caller sees, not internal state?
- A bug fix has a test that fails without the fix.

## Architecture
- Does it follow the repo's existing patterns? If it adds a new one, does it say why?
- Does a refactor remove concepts a reader has to hold, or just move them?
- Is feature logic leaking into a shared module, or is there a near-copy of a helper that already exists?
- Has a new condition been bolted onto an unrelated flow?
- Does it add a dependency? See [dependencies.md](dependencies.md).
- SQL, schema or migrations: use the database skill's checks if it is installed.

## Readability
- Can the change be read without the author explaining it?
- Is there dead code the change leaves behind: unused functions, imports, flags, `// removed` comments?
- Is anything clever that could be plain?

## Smells (judgement calls only)

Report these under Judgement calls as "possible <smell>", never as findings. The repo's rules win: suppress a smell the repo endorses. Also check the signals in [simplify.md](simplify.md).

- **Mysterious name**: the name doesn't say what it does or holds. Rename it; if no honest name comes, the design is unclear.
- **Duplicated code**: the same logic in three places, or two copies that have already drifted apart. Extract it on the third copy; two copies that agree stay inline.
- **Feature envy**: a function that uses another object's data more than its own. Move it to that data.
- **Data clumps**: the same few fields or parameters always travel together. Bundle them into one type.
- **Primitive obsession**: a string or number standing in for a domain concept. Give it its own small type.
- **Repeated switches**: the same switch or if-chain on the same value in several places. Use one map or one dispatch both share.
- **Shotgun surgery**: one change forces edits scattered across many files. Gather what changes together.
- **Divergent change**: one file edited for several unrelated reasons. Split it.
- **Speculative generality**: parameters, hooks or abstractions for needs the intent doesn't have. Delete them.
- **Message chains**: long `a.b().c().d()` walks a caller shouldn't depend on. Hide the walk behind one method.
- **Middle man**: a function or class that only passes calls on. Call the real target.
- **Refused bequest**: a subclass that ignores most of what it inherits. Use composition instead.
