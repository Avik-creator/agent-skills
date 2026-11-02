# Sources

Read on 2026-11-02. The downloaded folders have no git history, so no commit SHA is recorded; compare against upstream by file content.

| Origin | Read | Became |
|---|---|---|
| Two-axis `code-review` skill (pasted in chat; also installed at `~/.claude/skills/code-review`) | in full | SKILL.md step 1 (pin the base, fail early), the intent/rules split for two reviewers, checks.md smells |
| `requesting-code-review` (pasted in chat) | in full; its `code-reviewer.md` template was not available | the fresh-reviewer rule, pushing back with proof (Disputed) |
| `code-review-and-quality`, `~/Downloads/agent-skills-main/skills/code-review-and-quality/SKILL.md` | pasted copy in full; local copy's headings and ends checked against it | checks.md (five checks), proof.md (mutations, rule out via callers, pre-existing), dead code, dependencies.md |
| `agent-skills-main/references/security-checklist.md`, `performance-checklist.md` | in full | checks.md Security and Performance, dependencies.md audit and install scripts |
| `code-review-excellence`, `~/Downloads/agents-main/plugins/developer-essentials/skills/code-review-excellence/` (SKILL.md and both references) | in full | feedback.md wording (questions, optional marks), skip what linters catch |
| GitHub REST docs, create a review; GitLab MR docs | checked 2026-11-02 | feedback.md review JSON; the GitLab fetch ref in reviewer-brief.md |
| `code-simplification`, `~/Downloads/agent-skills-main/skills/code-simplification/SKILL.md` (MIT, Addy Osmani) | in full, 331 lines | references/simplify.md: signals table, over-simplification traps, the understand-first questions, one change at a time with unmodified tests, separate commit, revert if not clearer. Dropped: "delete what-comments, keep why-comments" (conflicts with the one-line what-comment rule), the 50-line / 5-line / Rule-of-500 thresholds (unmeasured), all TS/Python/React examples |
| Decisions made with the user in chat | - | scope, proof rule, report sections, labels, intent sources, when it runs, posting, two reviewers, after the report |

Dropped on purpose: team-culture advice (praise, rapport, review speed), Python and TypeScript snippets, change-size and description rules, the multi-model diagram, the `/setup-matt-pocock-skills` dependency, needing a spec file.

## Added 2026-11-02 (source sweep)
From wshobson/agents `review-agent-governance` (`ai-debt-detector`): resources opened and not closed on every path, and calls that don't exist in the locked dependency version.

## Renamed 2026-11-02
The skill was called `code-review` until 2026-11-02. It was renamed to `review` because the old name collided with Claude Code's built-in `/code-review` command.

## Added 2026-11-02: strict mode (references/strict.md)
From `thermo-nuclear-code-quality-review` in [cursor/plugins](https://github.com/cursor/plugins) `cursor-team-kit/skills/` (MIT, Copyright (c) 2026 Cursor). It was installed by the author, who folded it into review and removed it.
- **Taken:** the opt-in strict pass, deleting complexity before rearranging it, and the categories: spaghetti growth, one-off modes, edge cases mid-function, leaking, wrong home, blurred contracts, magic, temporary branches, orchestration. Also the order of remedies, fewer high-conviction comments, and a direct tone.
- **Dropped:**
  - The 1,000-line file rule and "presumptive blocker" on crossing it. The threshold is invented, and it conflicts with refactor's "never split for file size alone".
  - "Be ambitious, restructure the codebase, go for it". Here strict findings are proposals applied after a yes.
  - Pushing logic into a new abstraction, state machine or policy object by default. Here an abstraction is a remedy only when it removes branches or copies that exist now.
- **Changed:** a half-applied update that can actually happen is reported as a correctness Finding, not as Structure.
