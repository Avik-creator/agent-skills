# Exploring an idea before choosing a direction

Read when the idea is still open: the user says "ideate", "refine this idea", "what are my options", or describes a want without a chosen approach. Once a direction is picked, continue with interview.md.

## Goal
Turn a loose idea into one chosen direction, with its riskiest assumptions named and a way to check each one. Saying no to good ideas is part of the result.

## Steps
1. **Look first.** Read the repo, docs and tools that already exist. They are both a constraint and the cheapest material to build from.
2. **Restate the problem** as "How might we [outcome] for [specific user] without [constraint]?" No solution inside the question, and neither too broad ("make users happy") nor too narrow ("add a button to settings").
3. **Find the job.** "When I [situation], I want to [motivation], so I can [outcome]." Ask what people do today: the current workaround is the real competitor.
4. **Sharpen**, one question at a time, each with your recommended answer: who exactly, what success looks like, the real constraints (budget, time, team), what was tried before, why now. Don't offer variations until who and success are known.
5. **Offer 3-6 variations** in one turn. Each names the lens that produced it and why it exists, and you say which you'd pick. The user's choice is one decision.
6. **Group what the user liked** into 2-3 directions that differ in kind, not degree, and run the checks below on each.
7. **Recommend one direction**, and say why the others lose.

## Lenses for variations
Pick the ones that fit; never run them all.
- **Simplify or remove:** the version with the fewest steps, or only one feature.
- **Invert:** do the opposite, or move the work from the system to the user (or back).
- **Different user:** the same need for someone else, to see whose problem it really is.
- **Combine with what exists:** reuse a module, tool or process already in place.
- **Tight constraint:** built in one day, or at zero budget.
- **Structural analogy:** where a problem with the same shape was solved, for example "trust between strangers", not "Uber for X".

## Checks for each direction
- **Value:** who has the problem, how often, and what they do today. "Everyone could use this" means no one is named yet.
- **Feasibility:** the hardest part, dependencies we don't control, and how soon someone can use a first version.
- **Cost:** real numbers, searched when they are external facts, or marked "not known yet". Never guessed.
- **Assumptions**, in three groups:
  - **Must be true:** if wrong, the idea dies. Check these before building.
  - **Should be true:** if wrong, the approach changes but the idea survives.
  - **Might be true:** about extras. Don't check these yet.
  The first version exists to test the riskiest must-be-true assumption.
- **Pre-mortem:** "It shipped a year ago and failed. Why?" For each reason, say whether it can be prevented or means the idea should change.
- **Differentiation:** only when the thing competes for users, such as a product. Skip it for an internal feature or a process change.

Choose between directions with this grid, and use differentiation (when it applies) to break a tie:

| | Easy to build | Hard to build |
|---|---|---|
| **High value** | do first | worth the risk |
| **Low value** | only if trivial | don't |

## Ending
Be honest, not supportive: name a weak idea and say what makes it weak. Add the result to the decided list:
- the problem statement and the chosen direction, with its reason
- the directions dropped, each with its reason
- the assumptions by group, each with how to check it
- **not doing:** things someone might want now, each with its reason

Then offer interview.md to stress-test the direction, or a PRD or spec.

## Idea brief (only when asked)
Written after the decisions are settled, published only after asking where (SKILL.md, Deliver and publish). Sections: Problem, Chosen direction, Directions dropped, Assumptions to test, Smallest first version (what is in and out), Not doing, Open questions.
