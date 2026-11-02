---
name: prototype
description: Builds a throwaway prototype that answers one design question before real code is written - either a single double-click HTML file that lets anyone drive a state model through guided scenarios, or several structurally different UI variants on a real page, switchable with ?variant=. Keeps everything in a local .prototype/ folder, out of git, and turns the scenarios into the first tests when the answer moves into real code. Use when the user wants to sanity-check a state machine, business rule, workflow or data shape, asks "does this model handle X then Y", wants to click through a flow, wants to see a few layout options for a page or dashboard, or says "prototype this", "mock it up" or "let me try it first"; also when a design discussion stalls on a question talking can't settle. Not for production code or final visual design (use ui-ux-pro-max), or for writing the design document itself (use design-docs).
---

# Prototype

A prototype is throwaway code that answers one question. The question decides its shape, and the answer is the only part that survives.

## Quick start
1. Write the question in one sentence and pick the branch below.
2. Show the user a one-line plan (the question, the branch, where the files go). Build only after a yes.
3. Build it in `.prototype/<name>/` and check it runs before handing it over.
4. Hand it over, and change it while the user clicks through.
5. When it has answered the question, capture the answer and ask what to do with the files.

## When it runs
- When the user asks for one.
- When a design discussion (design-docs) reaches a decision that talking can't settle, suggest one in a single line, such as "a clickable demo of the retry states would settle this". Build only after a yes. Why: most decisions don't need a prototype, and one built unasked slows the discussion.

## Pick the branch

| The question | Read |
|---|---|
| Does this logic, state model, rule or data shape hold up? | [logic.md](references/logic.md) |
| What should this page or component look like? | [ui.md](references/ui.md) |

If it is unclear and the user is reachable, ask. If not, pick by the code it sits next to (a backend module means logic, a page means UI) and write the assumption at the top of the prototype. Why: the two branches build very different things, so a wrong pick wastes all of it.

## Where it lives
- Everything goes in `.prototype/<name>/` at the root of the project you are working in: the logic HTML, any database file, and `NOTES.md`.
- UI variants are the one exception. They live next to the page they render on, named `*.prototype.tsx` (or the project's extension), and `NOTES.md` lists their paths. Why: bundlers generally won't build code from a dot-folder outside the source tree.
- Keep both out of git with the local exclude file, not `.gitignore`:
  ```bash
  EX="$(git rev-parse --git-common-dir)/info/exclude"
  for p in '.prototype/' '*.prototype.*'; do grep -qxF "$p" "$EX" || echo "$p" >> "$EX"; done
  ```
  Why: it changes no file in the repo, and `git status` and code-review both skip excluded files. Not a git repo: skip this step.
- `NOTES.md` holds the question, the plan, every file outside `.prototype/` you touched, and later the verdict.

## Rules for both branches
- **Never touch real systems.** No production or shared databases, real third-party services, or real user data, not even to read. Why: a prototype is built fast and without guards, which is exactly when damage happens.
- **Made-up sample data only.** Never paste rows from a real system into a prototype. Why: the logic HTML is meant to be emailed around.
- **No persistence unless it is the question.** If it is, see [logic.md](references/logic.md#when-persistence-is-the-question).
- **Skip the polish.** No tests, no error handling beyond what makes it run, no abstractions. Why: the point is to learn fast, and the real code gets written properly later.
- **Show the state.** Render the full relevant state after every action (logic) or on every variant switch (UI), so the user sees what changed.
- **Easy to start.** Logic: one HTML file opened by double-click. UI: the project's normal dev command plus a URL.
- **Not code-reviewed.** Prototype files and the host page wiring are not real work, so skip code-review and say so at handover. It runs when the answer moves into real code.

## Capture the answer
When the prototype has answered its question:
1. Write the verdict and the question it settled in `NOTES.md`, and add it to the design-docs decided list or design write-up if there is one. Why: code-review checks real code against those decisions, so the review proves the code matches what the prototype showed.
2. Ask the user: keep the files on a local branch, or delete them? To keep them, run the recipe below, and commit only after that yes.
3. Undo any edit to a file outside `.prototype/` (the host page wiring), using the list in `NOTES.md`.
4. Comment on an issue or PR only when the user asks.

Keeping the files, without touching the current branch:
```bash
git diff -- <host page from NOTES.md> > .prototype/<name>/wiring.patch   # UI only; re-apply to re-run
WT="$(git rev-parse --show-toplevel)/.prototype/.keep-<name>"
git worktree add -b prototype/<name> "$WT" HEAD
mkdir -p "$WT/.prototype" && cp -R .prototype/<name> "$WT/.prototype/"
rsync -R <each *.prototype.* path from NOTES.md> "$WT"/
git -C "$WT" add -f .prototype/<name> <the same paths>
git -C "$WT" commit -m "prototype: <name> - <verdict in one line>"
git worktree remove "$WT"
```
The branch stays local. Push it only when the user asks.

## Moving the answer into real code
- The answer moves in, never the prototype code. Rewrite it properly.
- Write the tests first from the prototype's scenarios, watch them fail, then write the code. The details per branch are in each reference.
- Run code-review on the result before handing it over.
