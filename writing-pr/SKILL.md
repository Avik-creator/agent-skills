---
name: writing-pr
description: >-
  Use when writing or updating a pull or merge request title or body, adding
  screenshots or a diagram to one, or drafting replies to review comments.
  Not for pushing or opening the PR - use ship; not for reviewing someone
  else's PR - use code-review.
---

# Writing a PR or MR

## Scope

The title, the body, keeping the body current, and drafts of replies to review threads. Creating the PR or MR is `ship` step 5. Editing a body that is already up runs when the user asks, or after a yes to the shown text.

Replies to review threads are drafts. Never post a comment, a reply, or a reaction as the user unless they say so for that thread or that routine.

## Before writing

Read the whole diff against the target branch, the commits, and the request. Ask every blocking question in one turn. Never invent a fact. A number, a payload, or a screenshot comes from this change, not from memory.

If the diff mixes unrelated changes or is too big to review in one sitting, propose splitting it into separate or stacked MRs before writing anything.

## Title

Short. It says what changes, in the repo's existing title style; read recent merged titles first.

## Body

Diagrams and code come first, then bullets around them. No essays: any prose is a bullet.

1. **One line**: what changes, and for whom.
2. **Flow**: a simplified mermaid diagram of the path that changed. Stop at the zoom that answers "what does this change do", as the design-docs skill's `flows-and-diagrams.md` says. Leave it out only for a change with no flow, such as a typo or a version bump.
3. **Code**: a short snippet of the internals that matter, or sample usage. For an API change, the request a client sends and the response it gets, built from the real types.
4. **Changes**: bullets, each with a code reference where it helps.
5. **Before and after**:
   - Visual changes, direct or indirect: a table with before and after images or videos.
   - Benchmarks: a table with the baseline from the target branch and the candidate from this branch, with units and the command.
6. **Before merge and deploy**: only when there is something to do. The exact commands, environment variables, migrations, and their order, plus the undo for each one that changes data.
7. **Kept on purpose**: a suppression, skip, or lowered bar the user agreed to, with its reason (ship's bar-guard).

For a change that is hard, high-risk, or wide in scope, write the body like a short technical blog post: the context, the problem, the approach, code samples, before and after, diagrams, images.

## Never in the body

- "Validation", "I ran tests", test counts, or a list of commands run. Why: CI shows it, and the body is about the change.
- How the PR got here: earlier sizes, refactors between commits, rebases, review rounds. Why: only the final squash commit is read later.
- AI attribution: no "Generated with", no co-author trailer.
- Secrets, tokens, connection strings, internal URLs with credentials, or personal data in screenshots.
- Anything a reviewer wouldn't miss. When the body is long, cut sections before shortening sentences.

## Images

Upload them and embed the returned Markdown. Uploading is outward-facing, so show the files first and upload after a yes.
- GitLab: `glab api projects/:id/uploads --form "file=@<path>"`. The response's `markdown` field is the embed.
- GitHub: `gh` has no image upload. Ask the user to drop the image into the PR in the browser, then reference it.

## Updating the body

- After a push that changes behaviour, or after review feedback is addressed, rewrite the body so it describes the final state. Do not append an "update" section.
- Show the new text, then run:
  - GitLab: `glab mr update <iid> --description "$(cat <file>)"`
  - GitHub: `gh pr edit <number> --body-file <file>`
- Read it back after the update.

## Review replies

Per thread: the file and line, the reviewer's point in one line, and the drafted reply. A reply says what changed and where, or why not, with the evidence. The user posts it.

## Replies

Show the title and body as they will appear. Say what was left out and why.
