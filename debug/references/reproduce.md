# Reproducing a failure

Read at SKILL.md step 2. A cause you can't make fail on demand can't be proven, and a fix for it can't be told apart from luck.

## The repro spec
Fill this before forming any theory. A blank row is the next question for the user.

```
Build:       <version and commit SHA>
Environment: <OS, browser and version, device, CI or local>
Locale:      <locale, time zone, currency>   (when the result involves dates, money, sorting or text)
Data:        <the account, the input, the records involved>
Steps:       <the exact path, one action per step>
Expected:    <the value or behaviour that should happen>
Actual:      <what happened, with the exact message or number>
Frequency:   <every time, or N of M tries>
When:        <time of first occurrence, with time zone>
Evidence:    <the error text, log lines, response body, trace>
```

Why each row matters:
- **Build**: the report may come from a build where it is already fixed, or from a deploy no one else has.
- **Data**: a wrong result usually depends on the exact input, such as two items rather than three, a percentage coupon rather than a fixed one, or one account's settings.
- **Locale**: rounding, tax, date formats and sorting change with locale and time zone.
- **Frequency**: "sometimes" means something isn't fixed in place, so pin it before shrinking (below).
- **When**: line it up with deploys, config changes, scheduled jobs and the time of day.

## Pin what varies
Fix each of these in the scratch reproduction so it fails the same way every run:
- **Time**: a fixed clock at the failing instant. When the bug is time-dependent, try midnight, month end and a time-zone change.
- **Randomness**: a fixed seed, or a stubbed random source.
- **Network**: canned responses for outside services, and an error for any call that wasn't stubbed.
- **Locale and time zone**: set them explicitly (for example `TZ`, `LANG`).
- **Order and state**: run alone, then with the tests or jobs that ran before it.

Then run the reproduction several times and confirm every run gives the same result. The test that keeps this pinned is written later with `testing` (its unit.md covers fake timers).

## When it won't reproduce
Do not close it as "can't reproduce" and do not call it flaky yet. Find which case it is:

| Case | What shows it | Next step |
|---|---|---|
| Flaky | The same commit and environment give both results over many runs | Find what isn't pinned: time, randomness, network, a race, shared state |
| One environment | It fails only under one config (time zone, locale, OS, browser version, CI or local) and every time there | Match the user's environment, then shrink |
| One input | It fails only with the user's account or data | Get that data or a copy of its shape and replay it |
| Not reproduced | None of the above after matching the environment and data and repeating | Report what was tried, with the environments, the run counts and the data |

Say which case it is and the evidence that shows it. A guard test for a defect that won't reproduce on this build is the `testing` skill's call, and the report says that the defect itself was not confirmed.
