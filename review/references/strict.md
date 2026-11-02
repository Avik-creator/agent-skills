# Strict maintainability review

Read when the user asks for a strict, deep, harsh or "thermo-nuclear" review of code quality. The normal review still runs in full. This adds a structural pass on top: the reviewer reads this file too, and the report gets a **Structure** section between Findings and Couldn't prove.

## The stance
- "It works" is not the bar. A change that keeps behaviour but leaves the code harder to reason about is a structural problem, reported as one, not softened into "possible".
- Look first for the move that deletes complexity: a reframing in which branches, modes, flags, helpers or a layer disappear because the existing design already handles the case. Prefer it over a cleaner arrangement of the same complexity.
- A refactor that moves code around without reducing the number of concepts a reader must hold is not an improvement. Say so.
- Fewer, high-conviction comments beat a long list. Leave out cosmetic notes when a structural one exists.

## What to report under Structure
Each item cites the file and lines, and names what got worse in terms a reader can check: the branches added to a flow, the flag that now changes control flow, the module that now imports what it didn't.
- **Spaghetti growth**: a new conditional, special case or early exit added to a flow that already juggles several, or to a path unrelated to the feature.
- **One-off modes**: a new boolean, nullable mode or option that changes control flow for one caller.
- **Edge cases mid-function**: narrow handling dropped into the middle of a busy function instead of at its boundary or in the caller that needs it.
- **Leaking**: feature-specific logic in a shared or general-purpose module, or an implementation detail exposed through an API.
- **Wrong home**: logic placed outside the module that already owns the concept, or a bespoke helper where the repo has a canonical one.
- **Blurred contracts**: a new `any`, `unknown`, unchecked cast, or optional parameter that hides the real shape; a silent fallback that papers over an invariant nobody stated.
- **Magic**: a generic mechanism that hides a simple, fixed data shape.
- **Temporary branches**: "for now" code paths with no removal condition.
- **Orchestration**: independent steps run one after another for no reason, or related updates that can leave state half-applied. A half-applied update that can actually happen is a correctness Finding, not Structure.

## Remedies to propose, in this order
1. Delete: the layer, wrapper, mode or branch the existing design makes unnecessary.
2. Reframe the state or data model so the conditionals disappear, rather than gathering them in one place.
3. Move the logic to the module that owns the concept, or reuse the canonical helper.
4. Replace a condition chain with a typed model or one dispatch, when the same chain already exists more than once.
5. Separate orchestration from business logic.

A new abstraction is a remedy only when it removes branches or copies that exist in the code now; never for a need the intent doesn't have. Why: an abstraction for a guessed future is the over-engineering this pass is meant to catch.

## Limits
- No line-count or file-size threshold. A file that grows is reported only when the growth mixes unrelated reasons to change (divergent change, checks.md).
- Structure items are proposals. On your own work, apply one only after a yes; on someone else's PR, report only, as SKILL.md says.
- The tone is direct: name the problem and the simpler shape. Never rude, never "maybe consider".

## Report line
```
Structure
- <file>:<lines> - <what got worse, checkable>. Simpler: <the shape that deletes it>.
```
