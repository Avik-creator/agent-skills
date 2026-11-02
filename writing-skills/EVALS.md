# Testing a skill with and without

Read at SKILL.md step 7. The question is whether the skill changes what an agent does, not whether its text sounds right.

## Write the cases first
- 2-5 prompts a real user would type, each with:
  - **Must**: things the answer has to do or contain, each checkable (a command run, a file touched, a phrase, a refusal).
  - **Must not**: the mistakes the skill exists to prevent.
- Write them before reading any run. Why: criteria written after the answers bend to fit them.
- Add 2-3 near misses: prompts that share keywords but should not load the skill.
- Take criteria from mistakes you have seen an agent make, not from the skill's own wording.

## Run three arms
Same prompt, same model, a fresh agent each time:
1. **Bare**: no skill, no project instructions.
2. **Rules only**: the project's instructions without the skill.
3. **Skill**: the skill added.

If the bare arm already passes a case, the case has no teeth. Replace it.

## Grade
- A pass meets every must and no must-not. An error, an empty answer, or nothing checkable is "incomplete", never a pass.
- Every "yes" quotes the span in the answer that shows it. A yes with no quote is unverified.
- Grade without knowing which arm wrote the answer where you can: strip names and paths that give it away.
- Doing beats saying: an answer that explains the rule but doesn't apply it fails.
- For a skill that claims to catch bugs, plant a known fault in a copy and keep a healthy control. The control must pass, and the fault must fail one named check. The agent under test must not see the fault or the answer.
- One run is an example, not a rate. Repeat a case before calling a difference real, and keep the counts per case rather than one blended score.

## Report
Per case: the three verdicts with their quotes, and what the transcript shows the agent did. Then the changes to the skill, each tied to a failed case. Fix a failure by sharpening the reason in the skill, then rerun that case in all three arms.
