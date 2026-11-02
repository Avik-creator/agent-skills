# Sources

## The author's skill
The author's `writing-pr` skill was shared on 2026-09-07. The original text isn't in any surviving transcript. This version is built from the rules saved in a work project's memory note, which says it summarises that skill:
- No essays: a concise body, with bullets for any prose.
- Never write "validation" or "I ran tests".
- Lead with mermaid diagrams and code samples or snippets, either internals or sample usage. Code references are welcome.
- For a visual change, direct or indirect: a before/after table with uploaded images or videos.
- For benchmarks: a before/after table, with the baseline from the target branch and the candidate from the PR.
- No history of how the PR evolved. Only the final squash commit matters.
- A truly impressive, difficult, high-risk or wide change is written like a technical blog post.

## The author's corrections in later sessions (work-project transcripts)
- 2026-09-21: "MR descriptions should all have a proper flow diagram in a simplified manner."
- 2026-09-21: "In the PR's can you tell what the post request should look like from the frontend".
- 2026-09-21: commit in phases, and "we don't want to have a huge MR".
- 2026-10-12, 2026-10-14: screenshots go in the MR body.
- 2026-10-13: "this MR description is too much big, can we reduce it".
- 2026-10-16: deploy steps in the body: "mention the commands and all in the MR body properly".
- 2026-10-07, 2026-10-16, 2026-10-21: update the MR description after pushes and review.
- 2026-08-03 memory: no Claude trailer in commits or MR descriptions.
- 2026-10-14 memory note: review replies are drafts the user posts. Explicit exceptions are allowed per routine.

## Changed in ship
`ship` step 5 used to put "how it was verified (the commands run and their results)" in the description. That contradicts "never I ran tests", so the author's rule wins: the verification goes in chat, and ship now calls writing-pr.

## Verified 2026-11-02
- `glab mr update --help` (glab 1.107.0): `-d/--description`, `-t/--title`.
- `gh pr edit --help` (gh 2.96.0): `--body-file`, `--title`.
- GitLab docs (Project markdown uploads): `POST /projects/:id/uploads` with form field `file`. The response has a ready-to-embed `markdown` field. The page doesn't say who can view uploads in private projects, so uploading waits for a yes.
- `glab api --form` uploads a file with `@path`.
- Not verified: whether `gh` can upload images. No such command was found, so the skill asks the user to drop the image in through the browser.

## Added 2026-11-02: plain words (references/plain-words.md) and simple-English replies
From pstack's `unslop` and `technical-writing` in [cursor/plugins](https://github.com/cursor/plugins) (MIT, Copyright (c) 2026 Lauren Tan), read in full from the downloaded `plugins-main` folder.
- **Taken:**
  - the AI-vocabulary, fancy-"is", long-word and metaphor-noun lists
  - "say what it does, not how it feels"
  - the hedging, adverb, rule-of-three, "-ing" tail and synonym rules
  - the placement of "only" and "not", clear pronouns, whole sentences, and mixed sentence lengths
  - the formatting tells, link text, and chat phrases
- **Dropped:**
  - Diátaxis modes; `documentation.md` already picks the document type
  - the STE and Google style rules this skill already has (active voice, one idea per sentence, defined terms)
  - "no colons" and "no semicolons", which are too strict for reference text
  - the repo-specific notes (tabs, budget scripts) and the worked example
- `writing-pr/references/plain-words.md` and `design-docs/references/plain-words.md` are identical copies. Change both together.
- Every skill's Replies now says to talk to the user in simple English, at the author's request.
