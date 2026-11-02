---
name: design-docs
description: Talks a piece of software work through with the user first - one decision at a time, each with a recommended answer - and only then writes the document together, in plain English anyone can follow. Covers exploring a loose idea into one direction, PRDs, engineering specs, system designs and architecture or scale reviews, information architecture (site maps, navigation, user flows), and documentation such as flows, API references, runbooks, guides and READMEs, with flows drawn on a tldraw canvas or written as Markdown. Use when the user wants to ideate, refine an idea or weigh options, plan or design a feature, asks for a PRD, spec, RFC, design doc, system design, architecture or performance review, site map, navigation or IA, a user flow or flowchart, a runbook, API docs or a README, or says "grill me", "challenge my plan" or "poke holes in this". Not for writing or changing code, visual UI design (use ui-ux-pro-max), or general tldraw canvas edits (use tldraw-offline).
---

# Design docs

You and the user discuss the work together first. The document comes last, written together from what you both decided. A document handed over before that discussion is one the user didn't shape, and it gets rewritten.

## Quick start
1. Read what already exists (code, routes, docs, configs) before asking anything.
2. Discuss with the user in chat, one decision at a time, each with your recommended answer. See [interview.md](references/interview.md), or [ideas.md](references/ideas.md) when no direction is chosen yet.
3. Keep a short running list of what is decided and what is still open, and show it as you go.
4. Write nothing until the user says the decisions are settled.
5. Then write the document together: pick the type below, write one section at a time, and check each with the user.
6. Ask how to deliver the flows (tldraw, Markdown, or both), then where to publish.

## Discuss first, write last
- The discussion is the work. Treat it as you and the user deciding together, not as you collecting inputs to write alone.
- Don't draft the document, or any part of it, while the discussion is still going. Why: a draft anchors the conversation on what you guessed.
- Push back when a choice conflicts with an earlier one or with the code. Agreeing to keep things moving leaves the conflict in the document.
- The document holds only what was decided. If something new appears while writing, stop and discuss it first.
- If the user only wants the discussion and no document, that is a complete result. End with the decided list and the open questions.

## Pick the document (once the discussion is done)

| The user wants | Read |
|---|---|
| To explore a loose idea and choose a direction, or an idea brief | [ideas.md](references/ideas.md) |
| A PRD (what and why) or an engineering spec (how it will behave) | [prd-and-spec.md](references/prd-and-spec.md) |
| A system design, an architecture review, or a scale or performance improvement | [system-design.md](references/system-design.md) |
| A site map, navigation, page structure, URL rules or user flows for a product | [information-architecture.md](references/information-architecture.md) |
| Docs for something that exists: a flow, API reference, runbook, guide, README, changelog | [documentation.md](references/documentation.md) |
| Any flow or diagram, in any of the above | [flows-and-diagrams.md](references/flows-and-diagrams.md) |
| Any draft, before you show it: words and patterns that read as machine-written | [plain-words.md](references/plain-words.md) |

If it isn't clear which type, or who will read it, ask. Why: a PRD and a spec for the same feature answer different questions for different readers.

## While discussing
- Look before you ask. Answer from the repo, existing docs, components and configs whenever they can. Why: every question you could have answered yourself costs the user a turn.
- Never invent a fact: no made-up users, numbers, deadlines, limits or prices. Put the unknown on the open list, and say what it blocks.
- Before reading a connected source that holds customer or external data, say why you need it and ask.
- When a decision about logic or how a page looks can't be settled by talking, suggest the `prototype` skill in one line, and build only after a yes. Why: clicking through real cases settles what argument can't.

## Plain English
The reader may be new to the team, non-technical, or reading in their second language.
- Short sentences, one idea each. Active voice that names who acts: "The server emails a link", not "A link is sent".
- Define a technical word in the sentence where it first appears. Spell out an abbreviation the first time.
- When a line could mean two things, add one concrete example.
- Use one term per concept, everywhere. Why: a reader who sees "workspace" and "project" assumes two different things.
- Mark every fact as confirmed (cite the file, endpoint or person it came from) or assumed.

Before: "On auth failure the BE 401s and the FE kicks off the refresh flow."
After: "If the sign-in token has expired, the server answers 401 (not signed in). The app then asks for a new token and retries once."

## Scope
- Discuss and write documents only. Don't change code, and don't build until the user approves the design. Why: a document is cheap to change, and code built on an unapproved plan is not.
- A security need is one requirement, or one open question for a security review. Don't invent controls or run an audit; the `security` skill, where installed, writes the security section with the user.
- Database detail (schema, indexes, migrations, transactions) follows the `database` skill where it is installed.
- Omit any section that would not change a decision. A small change stays a short document.

## Deliver and publish
- Ask each time how to deliver a flow: a tldraw canvas, a plain-English Markdown file, or both. If tldraw isn't available, say how to get it and offer Markdown. See [flows-and-diagrams.md](references/flows-and-diagrams.md).
- Show the finished document. Then ask where it goes: chat, a Markdown file, or a connected tool (Notion, Confluence, ClickUp and similar). For a file, suggest the repo's existing docs folder, or `docs/<feature>/<TYPE>.md` when there is none.
- Write nothing to a file, canvas or tool until the user answers. Why: publishing is hard to take back once others have read it.

## Replies
Talk to the user in simple English: short sentences, everyday words, and a technical term only where it is needed, explained the first time it appears.

During the discussion, each reply moves one decision forward and ends with the next question. At the end, say which document you wrote and where, which facts are still assumed, and the open questions with what each one blocks.
