# Sources

`SKILL.md` and `TEMPLATE.md` were merged earlier from Anthropic's skill-creator and a third-party template the author follows; no record of those was kept here before 2026-11-02.

## Added 2026-11-02 (source sweep)
- `EVALS.md` from qa-skills-main `scripts/run_evals.py`, `scripts/judge_evals.py`, `docs/EVAL-JUDGE-2026-10-21.md`, `docs/QA-ORCHESTRATION-2026-10-21.md`, `docs/REAL-WORKFLOWS-2026-10-21.md`, `docs/WITH-WITHOUT-2026-10-22.md`, `docs/SKILL_TEMPLATE.md` (read by subagents): must and must-not written first, three arms (bare, rules only, skill), a bare pass means a weak case, quoted grades, incomplete is not a pass, a healthy control plus a planted fault, one run is not a rate, no blended score.
- The link check from qa-skills-main `scripts/validate_skills.py`.
- Plain-English rules from wshobson/agents `avoid-ai-writing`. Its numeric tells (em dashes per 1,000 words, type-token ratios, "5 to 20x") were dropped as unsourced.
- Dropped: plugin-eval's length-based quality score and badges, which would punish short skills.
