# agent-skills

Skills for coding agents: write code, test it, debug it, review it, refactor it, and ship it, with the rules each stage needs. Each skill is a folder with a `SKILL.md` under 100 lines, plus reference files that load only when needed.

Start with [which-skill](which-skill/SKILL.md) when a request could match more than one skill or needs several in a row.

## Skills

| Skill | What it does |
|---|---|
| [automation](automation/SKILL.md) | Drives a real browser or desktop app to test, check or automate anything that needs a real interface - opening pages, clicking through flows, filling forms, taking screenshots, confirming a feature works. |
| [`review`](review/SKILL.md) | Reviews a code change through a fresh subagent that proves each finding - a failing test, a reproduction or a mutation the tests miss for a bug, the quoted rule for a rule breach - and reports proven findings apart from unproven risks and opinions. |
| [coding](coding/SKILL.md) | Use when writing or changing code, including a test that was asked for. |
| [database](database/SKILL.md) | Designs, writes, reviews and changes database-backed application code safely - tenant-scoped, index-aware queries, keyset pagination, schema and constraint design, zero-downtime migrations, race-free transactions, connection pooling, and careful access to real databases - with detailed Postgres guidance. |
| [debug](debug/SKILL.md) | Use when asked to diagnose, debug, or find why something fails: a crash, a wrong result, an error in a log or CI run, a test that started failing, a failure that comes and goes, a regression, or an error in production. |
| [design-docs](design-docs/SKILL.md) | Talks a piece of software work through with the user first - one decision at a time, each with a recommended answer - and only then writes the document together, in plain English anyone can follow. |
| [go](go/SKILL.md) | Writes, changes and reviews Go code to current standards - version-correct idioms from the Modern Go Guidelines CLI, stdlib-first boring design, and a fixed set of checks before handover. |
| [observability](observability/SKILL.md) | Instruments a feature so on-call can tell that it broke, where time went, and why, using logs, metrics, and traces, and designs symptom alerts. |
| [prototype](prototype/SKILL.md) | Builds a throwaway prototype that answers one design question before real code is written - either a single double-click HTML file that lets anyone drive a state model through guided scenarios, or several structurally different UI variants on a real page, switchable with ?variant=. |
| [refactor](refactor/SKILL.md) | Audits a repo or an area of it for drift, duplication, divergent flows, reimplemented code, over-engineering, dead code and visible waste, then applies approved fixes one behaviour-preserving step at a time. |
| [security](security/SKILL.md) | Threat-models a change that crosses a trust boundary, turns each abuse case into a failing test, triages a scanner or advisory finding, and writes a design review into the design doc. |
| [ship](ship/SKILL.md) | Takes a finished change to an open pull or merge request, then stops so the user merges and releases. |
| [test-cases](test-cases/SKILL.md) | Use when asked for test cases, QA coverage, or how to test a flow by hand. |
| [testing](testing/SKILL.md) | Writes automated tests that hunt for errors - edge cases above all, plus logic, structure, state, error-handling and permission errors - and proves each test by making it fail. |
| [which-skill](which-skill/SKILL.md) | Picks the skill, or the short chain of skills, for a request that could match more than one or that needs several in order: a bug to diagnose and fix, a feature from spec to merge request, a release, a cleanup, a design. |
| [writing-pr](writing-pr/SKILL.md) | Use when writing or updating a pull or merge request title or body, adding screenshots or a diagram to one, or drafting replies to review comments. |
| [writing-skills](writing-skills/SKILL.md) | Writes and refines agent skills (a SKILL.md plus optional reference files and scripts) in one standard shape - a trigger-first description, a body under 100 lines, detail split one level deep, and a quick with/without test. |

## Install

Copy or link a skill folder into your agent's skills folder, for example for Claude Code:

```sh
ln -s "$PWD/debug" ~/.claude/skills/debug
```

Install the skills you want. Several hand work to each other by name (`debug` to `coding`, `ship` to `writing-pr`), and they say "where installed" when another skill is optional.

## Sources

`_sources/<skill>.md` records, for each skill, the upstream material it adapted, what was changed or dropped, and how its facts were checked. Third-party licences are in [NOTICE.md](NOTICE.md).

## Licence

MIT, see [LICENSE](LICENSE).
