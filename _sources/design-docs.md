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
