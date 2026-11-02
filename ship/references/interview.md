# The interview

Read before interviewing the user (SKILL.md step 4), and for a short re-interview after new commits.

The agent interviews; the user is the candidate. Each answer is checked against the code. The interview teaches; it never blocks the push. Why: the user is about to put their name on code they did not type, and should be able to explain it.

## Prepare (before the first question)
- Read the diff from the merge-base, and the code it calls and is called by.
- Write 6-10 questions, each with its answer and the proof: a `file:line`, or a command that shows it. Keep these to yourself. Why: you grade against the code, not against what you remember writing.
- Every question is about lines this change adds or modifies. No general trivia.
- Order them from easy to hard:
  1. **What**: what the feature does, and for whom.
  2. **Flow**: how a request or job moves through the changed code.
  3. **Why**: why it is built this way, and why the obvious alternative was not used.
  4. **Failure**: what happens on a timeout, a retry, a duplicate, a partial failure, a race, bad input. Pick only the ones that apply to this change.
  5. **Security**, when the change touches input, auth, files, secrets or tenants: who can call it, and what stops someone else's data leaking.
  6. **Undo**: how to turn it off or roll it back, and what happens to data it already wrote.

## Ask
- One question at a time. Wait for the answer.
- No hints, no multiple choice, no guess attached. Why: a hint gives the answer away.
- "I don't know" is a fine answer. Explain, and move on.
- The user can say "stop" at any point; go to the scorecard.

## Grade each answer
Reply with the grade, then the proof:
- **Right**: one line, and where the code shows it.
- **Partly**: what was right, what was missing, and where the code shows it.
- **Wrong**: the real behaviour in two or three lines, where the code shows it, then one follow-up question on the same point. Why: the follow-up shows whether the gap closed.
- **The user is right and the code is not**: this is a bug, not a wrong answer. Say so plainly, and fix it after a yes before anything is pushed. A fix that changes a schema, an API, security behaviour or what users see is asked about first.
- Run proof commands only against local or test setups, never production.
- If the user disputes a grade, check again and show the proof. Change the grade when they are right.
- Be honest. No praise padding: "Right" is enough.

## Scorecard
```
Interview on <branch> at <short sha>: <n> questions.
Right <n> · Partly <n> · Wrong <n> · Code bugs found <n>
To revisit:
- <topic>: <one line>, see <file>:<line>
```
The scorecard stays in the chat. It does not go in the commit or the PR or MR.

## Short re-interview (new commits)
When the hook blocks because of new commits, ask 2-3 questions about only the lines those commits added or changed (`git diff <recorded head>..HEAD`). Same grading. Then update the record.
