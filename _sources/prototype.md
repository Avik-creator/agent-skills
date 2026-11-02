# Sources

Read on 2026-11-02.

| Origin | Read | Became |
|---|---|---|
| `prototype` skill, [mattpocock/skills](https://github.com/mattpocock/skills) `skills/engineering/prototype/` (SKILL.md, LOGIC.md, UI.md, agents/openai.yaml), MIT, commit `321658273cb1d20b76026717d027d505790106d4` (2026-08-19) | pasted in chat in full; SKILL.md and UI.md checked against upstream, LOGIC.md first lines checked | the two branches, the logic HTML layout (question, state panel, free play, walkthrough tabs), the pure-module shapes, the UI ?variant= pattern and switcher, capture as a primary source |
| Decisions made with the user in chat | - | when it runs (asked, or suggested by design-docs after a yes), capture (verdict to the decided list; files on a local branch only after a yes; nothing outward), Go port via table-driven tests from walkthroughs, safety lines and the throwaway DB (container or `.prototype/` file, no /tmp), plan-first and a browser check for UI, `.prototype/` at the project root kept out of git with `.git/info/exclude` |

Dropped on purpose: auto-commit to a throwaway branch and the pointer on the implementation issue (both need a yes now), the `pnpm <name>` task-runner rule (UI uses the normal dev command), "skip the polish" for the fold-in (it is tests-first now).

Refresh: `gh api 'repos/mattpocock/skills/commits?path=skills/engineering/prototype&per_page=1'`, then diff from the SHA above.

## Added 2026-11-03
- "When it runs" now names `deliver`, which prototypes screens and logic by default. Decided with the author; the rule outside `deliver` is unchanged.
