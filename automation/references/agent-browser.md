# agent-browser

Read when using agent-browser (step 2): a separate, clean Chrome that it opens itself.

## Load its guide first

Before the first command in a session, load the guide that ships with the installed version, and follow it:

```bash
agent-browser skills get core          # workflows, common patterns, troubleshooting
agent-browser skills get core --full   # adds the full command reference and templates
```

Why: the CLI serves instructions that match its version. Anything copied here goes out of date with the next release, so this file holds only when to use what.

## The loop

Take a snapshot of the page, act on an element by its `@eN` reference, then take a snapshot again to check the result. Why: references come from the accessibility tree, so clicks don't depend on screen coordinates or pixel matching. The exact commands and flags are in `skills get core`.

## Specialised guides

Load one when the task fits it:

| Task | Command |
|---|---|
| An Electron desktop app (VS Code, Slack, Discord, Figma and similar) | `agent-browser skills get electron` |
| Exploratory testing, QA, or a bug hunt | `agent-browser skills get dogfood` |
| Recording network traffic to build an API client for a site | `agent-browser skills get derive-client` |
| Slack workspace automation | `agent-browser skills get slack` |
| A protected Vercel deployment | `agent-browser skills get protected-vercel-deployments` |
| Running inside Vercel Sandbox, or AWS Bedrock AgentCore cloud browsers | `agent-browser skills get vercel-sandbox`, `agent-browser skills get agentcore` |

`agent-browser skills list` shows everything the installed version has.

## Notes

- Its Chrome starts clean: none of the user's sign-ins. For a site that needs one, use its sessions or saved state as `skills get core` describes, and stop for passwords and two-factor codes as SKILL.md says.
- An Electron app is a web app inside a desktop window. Try `skills get electron` before falling back to desktop control.
- Its dashboard runs on port 4848 (`https://dashboard.agent-browser.localhost` when proxied). Stay on the dashboard's address; it proxies the session tabs, so session ports don't need exposing.
