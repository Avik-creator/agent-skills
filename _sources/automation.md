# Sources

Checked on 2026-11-02. The tools' own guides are loaded at run time, never copied.

| Origin | Version seen | Became |
|---|---|---|
| browser-use skill (pasted in chat) and the browser-use README, https://github.com/browser-use/browser-use | PyPI `browser-use` 0.13.14 | references/browser-use.md: local Chrome rules, accessibility-tree clicking, the install command |
| `remote-browser` skill (pasted in chat) | same CLI | references/browser-use.md, cloud browser section |
| agent-browser discovery stub (pasted in chat), https://github.com/vercel-labs/agent-browser | npm `agent-browser` 0.41.1 | references/agent-browser.md |
| agent-desktop README, https://github.com/vercel-labs/agent-desktop | npm `agent-desktop` 1.0.2 | references/agent-desktop.md |
| Decisions made with the user in chat | - | SKILL.md: tool order, ask before desktop control, safety limits, the user installs tools, the report |

Notes: the install guide linked from the pasted skill (`browser-harness/install.md`) names the older `browser-harness` command. The current CLI is `browser-use`, installed with uv, then `browser-use skill install`.
