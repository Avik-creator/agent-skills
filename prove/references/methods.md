# Ways to drive and check a surface

Read when picking how to check a surface. Rewrite "catches" and "misses" in this repo's terms before showing them to the user. Most methods drive a surface; parity and snapshots are also answer keys.

| Method | Catches | Misses | Cost |
|---|---|---|---|
| **Browser, driven like a user** (Playwright script or the `automation` skill) | dead buttons, state that doesn't refresh, client errors, a hidden control | whether data really saved; server errors the page swallows | a browser install once; 10-60 s a run; flakier than the rest |
| **HTTP calls** (`curl` or a small script) | wrong status codes, bad validation, missing fields, auth holes, contract drift | what the page does with the response | near zero; under 1 s |
| **CLI or TUI in a real terminal** (tmux or a PTY, on its own socket) | wrong output, wrong exit code, broken argument parsing, bad error text | output that came from stale or cached state | low |
| **Stored-state check** (SQL, the file, the bucket, the cache key) | "saved" on screen with nothing written, partial writes, duplicates | whether a user can see it; pair it with a drive | low with a local store |
| **Log and error scan** during the drive | errors turned into a 200 or a blank screen, retries, slow queries | logic bugs that log nothing | near zero |
| **Side-effect capture** (local email catcher, webhook listener, queue tap) | "sent" that never sent, wrong recipient, wrong payload | real delivery by the provider | medium setup; cheap per run |
| **Snapshot or pixel diff** against a saved good result: assert equal dimensions, then compare decoded pixels (pixelmatch or similar), never file bytes (the same pixels can encode to different bytes) | any change from the known good state | what the snapshot never covered | low once the baseline exists |
| **Parity** with an old system or a sibling surface | drift in a port or refactor; a new surface that disagrees with the trusted one | bugs both share | the old system runs beside the new one; about twice the drive |

## Making checks catch real bugs
- Use more than one record: a check that runs against a single row misses lists, sorting, paging and off-by-one errors.
- Check the stored state after every write, not only the screen or the 200.
- Name every side effect a flow must cause (email, webhook, queued job). A missing one is a failure, unless the plan marks it optional.
- Prefer a script over an interactive session for the core flows. A script can be run again and read by a reviewer.
- Don't run `npx playwright install` blindly: it can delete browser builds other projects use. Install into a git-ignored folder and pin a version that matches a browser already in the cache.
