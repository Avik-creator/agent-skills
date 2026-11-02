---
name: automation
description: Drives a real browser or desktop app to test, check or automate anything that needs a real interface - opening pages, clicking through flows, filling forms, taking screenshots, confirming a feature works. Also checks the agent's own work in the real interface after a change, when the user asks it to or the testing skill needs it. Uses browser-use (the user's own Chrome) first, then agent-browser (a separate, clean Chrome), then agent-desktop for desktop apps only after asking. Use when the user asks to automate a web or desktop task, test or check something in the browser, verify or check your own work, click through a flow, reproduce a UI bug, take a screenshot of a page, or fill a form. Not for writing test files that run again (use testing), or fetching a public page or API that a plain request can read.
---

# Automation

Use it to automate a task in a real interface, or to check your own work there when the user asks: open the page or app, do what a person would do, and report what happened. A plain request is cheaper: if curl or a fetch tool can read it, don't open a browser.

## Which tool, in order

1. **browser-use** - attaches to the user's own Chrome, with their sign-ins. With no usable local Chrome, or for tasks that must run in parallel, it can start a Browser Use Cloud browser, after asking. See [browser-use.md](references/browser-use.md).
2. **agent-browser** - opens its own clean Chrome. Use it when browser-use is missing or can't connect. See [agent-browser.md](references/agent-browser.md).
3. **agent-desktop** - for native apps, or when neither browser tool works. Always ask first:
   "Do you want me to control apps on your desktop? I can see and click anything on screen, but I won't take over your mouse unless I ask again."
   Ask every task. A yes doesn't carry over to the next one. Ask again before `--headed` (real mouse and keyboard). See [agent-desktop.md](references/agent-desktop.md).

Say why each time you move down the list: "browser-use can't reach Chrome, so I'm switching to agent-browser."

## Missing tool

Don't install anything. Show the command, and let the user run it if they want:

| Tool | Install |
|---|---|
| browser-use | `uv tool install --python 3.12 --upgrade browser-use`, then `browser-use skill install` |
| agent-browser | `npm i -g agent-browser && agent-browser install` |
| agent-desktop | `npm install -g agent-desktop`, then `agent-desktop permissions --guide` on macOS |

If they'd rather not, move to the next tool.

## Limits

The user's Chrome holds their real accounts: email, code hosting, cloud consoles. Act as a careful guest in it.

- **Your tabs only.** Open your own tabs, never touch or close one you didn't open, and close yours when done.
- **Stop at sign-ins.** Hand over at any password, two-factor code, consent screen or "which account?" choice. Using a sign-in Chrome already has is fine.
- **Ask before it counts.** Before anything that sends, posts, buys, deletes, merges, invites, or changes settings or data, say what you're about to do and wait for a yes. This includes submitting a form on a real site. Why: these can't be taken back, and a click is enough to do them.
- **Production is read-only.** On a live site, look but don't change anything unless the user says otherwise for this task. Make changes on localhost or staging.
- **Ask before anything that costs money**, such as starting a cloud browser.
- **Private content stays private.** Screenshots and page text can show messages and personal data. Don't paste them into public places (issues, pull requests, shared logs).

Each yes covers that one action. Ask again for the next one.

## Report

Keep it short, in chat:

- **Result:** what was checked, and whether it passed or failed, one line each.
- **Each failure:** the step, what you expected, what you saw, one screenshot, and any console errors.
- **Tool:** which one you used, and why you moved down the order if you did.
- **Leftovers:** test data you created, or anything you couldn't close or undo.

Screenshots are for failures only. Why: screenshots of passing steps cost tokens and bury the one that matters.

When checking your own work, test what the user will do, not only the line you changed. A dead click, a silent no-op or a stuck spinner is a failure, even when the code "works".

## Example

User: "Check the signup form I just fixed."
1. `browser-use --doctor` reports that Chrome isn't reachable. Say so, and switch to agent-browser.
2. Open `http://localhost:3000/signup`, submit it empty, then with a bad email, then with valid details.
3. Report: "Empty: shows 'Email is required' - pass. Bad email: no message, button stays disabled - fail (screenshot). Valid: account created - pass. Used agent-browser because browser-use couldn't reach Chrome. Left behind: test account test+1@example.com."
