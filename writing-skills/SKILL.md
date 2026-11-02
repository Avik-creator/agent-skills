---
name: writing-skills
description: Writes and refines agent skills (a SKILL.md plus optional reference files and scripts) in one standard shape - a trigger-first description, a body under 100 lines, detail split one level deep, and a quick with/without test. Use when the user wants to create a skill, turn a workflow, habit or repeated correction into a skill, edit or tighten an existing SKILL.md, or fix a skill that does not trigger. Not for running formal benchmarks or description-optimization loops - use skill-creator.
---

# Writing Skills

A skill works only if its description makes the agent load it at the right moment, and its body then changes what the agent does. Every rule below serves one of those two.

## Quick start

1. Copy [TEMPLATE.md](TEMPLATE.md) to `<skill-name>/SKILL.md`.
2. Write the description first.
3. Fill the body and stop at 100 lines.
4. Test with 2-3 real prompts, then run the checklist.

## Workflow

### 1. Capture intent
- Mine the conversation, memory and past corrections before asking anything; a correction the user repeated is the strongest rule you have.
- Ask only what is left: what it does, when it triggers, what the output looks like, what it is not for.
- Restate the intent and get a yes before drafting; a wrong restatement is cheap, a wrong skill is not.

### 2. Pick the right home
- **Skill**: a workflow started on demand (write a PR, review a diff).
- **CLAUDE.md**: a rule that must hold every session (comment style); a skill may never load for it.
- **Hook**: a rule that must never break (no auto-commit); instructions can be ignored, hooks cannot.

### 3. Write the description
The agent sees only name + description when choosing; the body is invisible until then.
- `name`: lowercase, hyphens, max 64 chars, matches the folder.
- Third person, max 1024 chars.
- Sentence 1: what it does, concretely.
- Sentence 2: `Use when ...` with the phrases and contexts users really type, including asks that never name the skill; agents undertrigger, so lean broad.
- Sentence 3 (if a neighbour exists): `Not for X - use Y.`
- All "when to use" lives here, never in the body.

Good: `Extract text and tables from PDFs, fill forms, merge files. Use when working with PDFs or the user mentions forms or document extraction.`
Bad: `Helps with documents.` (nothing to tell it apart from any other skill)

### 4. Write the body
- Imperative voice, with a one-line why beside each rule; the agent generalises from reasons but follows a bare MUST literally.
- Concrete examples (input -> output, before -> after) beat abstract advice.
- One term per concept, used everywhere.
- Nothing time-sensitive (versions, prices, "currently"); if unavoidable, put it in a reference file someone updates.
- Write for every future prompt, not just the examples you drafted with.
- Delete any line the agent would follow anyway; it only costs context.
- Nothing that would surprise the user if read aloud.
- Plain English: state the claim directly (not "it's not X, it's Y"), cut clauses the sentence doesn't need, and add no number, source or stake the facts don't have.

### 5. Split when it grows
- Body past 100 lines, distinct domains, or rarely needed detail -> move it to a reference file.
- Link each file from SKILL.md with when to read it: `See [references/go.md](references/go.md) when the code is Go.`
- References stay one level deep; a file over 300 lines gets a table of contents.
- One domain per file, so the agent reads only the one it needs.

### 6. Add scripts only when earned
Add `scripts/` when a step is deterministic (validate, format), when every test run rewrote the same helper, or when errors need exact handling. Scripts run without entering context, so they save tokens and drift.

### 7. Test
Follow [EVALS.md](EVALS.md): cases with must and must-not written first, three arms, quoted grades.
- Write 2-3 prompts a real user would type: casual, specific, with paths and context.
- Run each with and without the skill (one subagent each, same turn) and compare with the user.
- Read the transcripts, not just the outputs; cut any section that sent the agent down a wasted path.
- Try 2-3 near-miss prompts that share keywords but should not load the skill.
- Fix a failure by sharpening the reason, not by adding a MUST for that one case; rerun until the user is happy.

## Review checklist
- [ ] Description: third person, what + `Use when ...`, max 1024 chars
- [ ] Neighbour skill named with `Not for ... - use ...` where one exists
- [ ] SKILL.md under 100 lines
- [ ] References one level deep, each linked with when to read it; every link resolves and every file under `references/` is linked
- [ ] Every rule has a one-line why; no caps-lock MUSTs
- [ ] At least one concrete example
- [ ] Consistent terms, nothing time-sensitive
- [ ] Tested with and without the skill, including near misses
- [ ] A Replies line tells the agent to talk to the user in simple English
- [ ] Every line earns its place

## Replies

Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.
