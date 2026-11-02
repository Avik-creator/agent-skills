# Interviewing and stress-testing a plan

Read at the start of any design work: this is how you and the user discuss it together before any document exists. "Grill me" is the same method, pushed harder. If no direction is chosen yet, start with ideas.md.

## Goal

Reach a shared understanding of the plan. Every decision is either resolved or written down as an open question. The discussion produces decisions, not a document. Offer to write the document together only once the user says the decisions are settled.

## How to grill

- Map the decision tree first: the big choices, and which choices depend on others. Start with the decisions that others depend on. Why: answering a leaf first gets undone when its parent changes.
- Ask one branch at a time: one question, or a few that are tightly linked.
- Give your recommended answer with every question, and one line of reasoning. Why: the user can say "yes" in one word, and your reasoning gives them something to push against.
- Look before you ask. If the codebase, existing components, styles, design tokens, docs or data can answer it, read them and state what you found instead of asking.
- Push back when an answer conflicts with an earlier answer, with the evidence, or with the stated goal. Name the conflict plainly, and ask which one wins.
- Ask about the unhappy paths: empty states, failures, wrong input, missing permission, and what happens at ten times the expected use.
- Keep going until every branch is resolved or parked. "Relentless" means no open branch is skipped, not that settled answers get asked again.

## Branches worth walking

Use the ones that apply:

- **Who and why:** who uses it, what problem it solves for them, and how we will know it worked
- **Scope:** what is in, what is out, and what a reasonable person would assume is in but isn't
- **Main path:** the steps from start to finish, and the one screen or step where most of the time is spent
- **Data:** what is stored, who owns it, and what must never be lost
- **Failure:** what breaks, what the user sees when it does, and how they recover
- **Constraints:** budget, deadline, team size, what already exists and must not change
- **Later:** what this makes easier later, and what it rules out
- **Proof:** for each goal, where it is seen, what right looks like and who wrote that, and what result counts as a pass (see proof.md)
- **Pieces:** how the work splits into parts that can each ship and be checked alone, with dependencies pointing one way

## Ending

Summarise in plain English:

- the decisions made, each with a one-line reason
- the open questions, each with what it blocks
- the assumptions still unconfirmed

Then offer the next document: a PRD, a spec, a design, or an information architecture.

## Example

User: "Grill me on a team metrics dashboard."

Good first question: "Who opens this dashboard most: managers checking a weekly trend, or engineers checking today? I'd guess managers, weekly. That makes it a slow-changing summary page, not a live view, so it can be cached and refreshed every hour. Right?"

Bad first question: "What colour scheme do you want?" (A leaf decision, asked before the decisions it depends on.)
