# Feedback

Read before posting a review on someone else's PR, which happens only after the user says yes (SKILL.md step 6).

## Before posting

1. Show the user the exact comments, each with its file and line.
2. Wait for a yes. Each yes covers this one post.
3. Check the comments for private content: secrets, tokens, personal data, internal hostnames, or screenshots from logs. Why: a PR comment is public to everyone who can see the repo, and stays in its history.

## Post as one review

Post one review with comments on the lines, not a run of separate comments. Why: each separate comment sends its own notification.

On GitHub, write the review to a file and send it:

```json
{
  "event": "COMMENT",
  "body": "<summary: what was checked, what ran, how many findings>",
  "comments": [
    { "path": "src/webhook.go", "line": 58, "side": "RIGHT", "body": "<comment>" }
  ]
}
```

```bash
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input review.json
```

- `event` stays `COMMENT`. `APPROVE` and `REQUEST_CHANGES` only when the user asks. Why: they can unblock or block a merge.
- Comment only on lines the diff shows; GitHub can reject a review with a comment on any other line. Put anything else in `body`.
- For another host, use its own CLI or API, and check its docs for the review format first.

## How to word it

- **Findings**: say what breaks and when, then the proof, then a fix if there is an obvious one. "A retried delivery writes a second row: `TestRetryTwice` gets 2 rows, want 1. An idempotency key on the delivery id would stop it."
- **Couldn't prove**: ask, don't claim. "What happens if two workers pick up the same job? I couldn't test it here with SQLite."
- **Judgement calls**: mark them optional. "Optional: `uc` could be `userCount`. Not blocking."
- Write about the code, not the person: "this function", not "you".
- Keep it short. One finding per comment, and no praise added for balance. Why: the author needs to see what to act on.

## When the author replies

- Facts and proof outrank opinions. If they show a test or a fact that rules a finding out, re-run your proof. If it no longer holds, say so and withdraw the finding.
- If they know something about the system you don't, defer, and tell the user.
- Answer only with the user's yes for each reply, the same as the first post.
