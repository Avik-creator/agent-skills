# Sources

Merged on 2026-11-02 from four skills the user pasted in chat. Their upstreams are unknown, so none is listed here. The ideas were rewritten, not copied wholesale.

| Origin | Became |
|---|---|
| `plain-paper` (PRD, spec, documentation) | SKILL.md shared rules, prd-and-spec.md, documentation.md |
| `the-map` (system design: as-is, proposal, improvement) | system-design.md |
| `information-architecture` (site map, navigation, IA template) | information-architecture.md |
| `grill-me` (interview until shared understanding) | interview.md |
| addyosmani/agent-skills `skills/idea-refine` (MIT, Addy Osmani 2025; zip pulled 2026-11-02, no SHA recorded) | references/ideas.md: HMW restatement, JTBD, six of its lenses, value/feasibility/assumption-audit/pre-mortem checks, the value x feasibility grid, idea-brief sections. Dropped: the Jobs-quote philosophy, "budget no factor", "10x version" and "expert lens" lenses, full SCAMPER and first principles, the startup-only differentiation rubric (kept only for products), 3-5 questions at once, the one-pager written mid-session, `scripts/idea-refine.sh` (only a mkdir), and examples.md (238 lines, invented figures). Added: cost as a check |
| Author's working rules (verify facts on the web, cost as a requirement, design in layers, measure before optimising, failure-mode list, what each choice enables later, plain English) | system-design.md, SKILL.md |
| tldraw offline app (https://offline.tldraw.com) and its `tldraw-offline` agent skill | only referenced, never copied. The app installs and updates that skill itself, through Develop → Install Agent Skills |

## Added 2026-11-02 (source sweep)
From wshobson/agents `incident-response/incident-runbook-templates`, `documentation-standards/grounded-vault`, `c4-architecture`, and addyosmani/agent-skills `documentation-and-adrs`, `spec-driven-development`: runbook preconditions and what to do when a command fails; numbers must be in the cited source; keep qualifiers when editing; stop a diagram at the zoom that answers the question; split work into pieces that ship alone.

## Added 2026-11-02: screening the shape (references/system-design.md)
From pstack's `architect` skill and its `references/design-red-flags.md` in [cursor/plugins](https://github.com/cursor/plugins) (MIT, Copyright (c) 2026 Lauren Tan), read in full from the downloaded `plugins-main` folder.
- **Taken:**
  - the eight red flags: shallow module, information leakage, temporal decomposition, pass-through method, split ownership, two ways to do one task, importable internals, hand-synced list
  - designing it twice, with structurally different candidates
  - preferring the smaller interface when two shapes pass
  - the assumption that the next contributor sees only one file and copies the nearest example
- **Dropped:**
  - the multi-model `arena` synthesis and the Cursor model lines
  - sketching `not implemented` code, and implementing straight after with no checkpoint. Here the design is a document, and building waits for the user's approval.
  - the "scrap the sketch" phase, which belongs to coding rather than to a design document

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

## Added 2026-11-03: proof for every goal (references/proof.md)
From Ray Amjad's `idea-to-spec` and the Prove It README in ray-amjad/skills (MIT, Copyright (c) 2026 Ray Amjad), and pstack's `poteto-mode/playbooks/visual-parity.md` in [cursor/plugins](https://github.com/cursor/plugins) (MIT, Copyright (c) 2026 Lauren Tan), read in full from the downloaded `skills-main` and `plugins-main` folders.
- **Taken:**
  - surface, answer key, check and pass signal for every goal, decided by the user
  - the five-rung answer-key ladder, and which key fits a new flow, an existing flow, a copied surface and a migration
  - "a fully mocked test proves no goal" and "the builder reviewing its own work is not a verifier"
  - human-judgment checks need a proposed replacement and never count as proof; a missing environment is a question, never a weaker check
  - 0% pixel difference for parity work (pstack); a user-chosen threshold for a new screen, at the author's request
- **Dropped:**
  - idea-to-spec's fresh-agent critique of a draft, its GitHub-issue publishing default and its invariants and ADR workflow; this skill already discusses first and asks where to publish
  - the course links and beta notes

## Changed 2026-11-03, found in a test run by a Cursor agent on a real feature (a blog tag filter)
- proof.md: a list's pass signal is equality; a key computed by the code under test is not a key, so expected results come from a standalone script saved before the lock; new UI on an existing screen is compared by element screenshots of the unchanged parts at 0%, not a whole-page threshold.
