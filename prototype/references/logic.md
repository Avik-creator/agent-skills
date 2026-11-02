# Logic prototype

Read when the question is about business logic, state transitions or data shape: something that looks fine on paper and only feels wrong once real cases go through it. You build one self-contained HTML file at `.prototype/<name>/index.html` that anyone can open by double-click and drive by pressing buttons, including a designer, a PM or a domain expert.

## 1. State the question
Write the question in one paragraph at the top of the page, in a visible intro and not a comment. Example: "Can an order be refunded after it was partly shipped, and what happens to the shipped items?" Why: a prototype that answers the wrong question is wasted, and a visible question can be checked later.

## 2. Write the logic as a pure module
Put the logic that answers the question in its own `<script>` block, as a small pure module. Pick the shape that fits the question:

| Shape | Fits when |
|---|---|
| A reducer, `(state, action) => state` | actions are discrete events and state is one value |
| A state machine with explicit states and transitions | which actions are legal right now is part of the question |
| A few pure functions over a plain data type | there is no current state, only transformations |
| A small class with a clear set of methods | the logic really owns ongoing internal state |

- Keep it pure: no `document`, no DOM, no click handlers inside it. The page calls the module; nothing goes the other way. Why: a pure module can be read on its own as the spec, and lifted or ported later.
- Illegal actions return a clear refusal (such as `{ok: false, reason: "already shipped"}`) rather than doing nothing. Why: a silent no-op looks like a broken button, and the refusal is often the answer.
- Write it in the language the real code will use where you can. If the real code is JS or TS, write the module in that style so it lifts straight in. If it's Go or anything else that isn't JS, the module is still JS (it has to run in a browser), and it becomes the reference for the port in step 6.

## 3. Build the page
One file, plain HTML, CSS and JS, everything inline: no framework, no bundler, no server, no CDN links. Why: it has to open by double-click and survive being emailed.

Write every label in the language of the business, not the code: "Ship 2 of 3 items", not `dispatch({type:'SHIP_PARTIAL'})`. Lay it out top to bottom:

1. **Title and the question**, in one or two lines.
2. **Current state**: every relevant field as a labelled panel, not raw JSON, redrawn after every click. Call out what the last action changed, such as "Refundable amount: 30 → 10".
3. **Free play**: one button per action, always visible, usable in any order. A refused action shows its reason.
4. **Guided walkthroughs**, one tab per scenario. Each tab has a short plain description (the situation and what to watch for), then the ordered steps as real buttons. Clicking a step performs it and moves to the next. Starting a walkthrough resets to a fixed starting state, so it runs the same every time.

Pick walkthroughs for the awkward cases: the happy path, a tricky edge case, and an attempt at something that should be illegal. Each walkthrough also states the expected end state in plain words. Why: those become the first tests in step 6.

Keep it plain and calm: clean type, generous spacing, one accent colour, no animation.

## 4. Check it, then hand it over
- Open the file and click through every walkthrough yourself before handing it over (use the automation skill if it's installed). Why: the user shouldn't be the first to find a dead button.
- Give the user the path, or open it for them. The useful moments are "wait, that shouldn't be possible" and "I assumed X would be different": those are the bugs in the idea.
- Add actions or walkthroughs when they ask. Each new case they raise usually becomes a walkthrough.

## When persistence is the question
Only when the question is about storage itself, such as "does this schema let me represent X?":
- **A file is enough**: a SQLite file at `.prototype/<name>/PROTOTYPE-wipe-me.db`.
- **A real engine is needed** (Postgres behaviour, constraints, locking): a fresh container with no volume mount, bound to localhost, named after the prototype:
  ```bash
  docker run -d --name prototype-<name> -e POSTGRES_PASSWORD=prototype -p 127.0.0.1::5432 postgres:<the project's major version>
  docker port prototype-<name> 5432   # the local port it got
  docker rm -fv prototype-<name>      # when done; -v also removes its anonymous volume
  ```
- Never reuse an existing container or database, even a local one. Why: an existing one may hold copied real data.
- Use the database skill, where it is installed, for the schema and queries.
- Write down in `NOTES.md` what was created, and remove it when the prototype is deleted.

## 5. Capture the answer
Follow SKILL.md "Capture the answer". The walkthroughs and their expected end states go in `NOTES.md` with the verdict.

## 6. Move the answer into real code
The walkthroughs become the first tests. Write them before the code, and watch them fail.

- **JS or TS:** lift the pure module into the real code and turn each walkthrough into one test case, with the steps and the expected end state.
- **Go, or any language that isn't JS:** port the module, with the tests first. Each walkthrough becomes one row in a table-driven test:

```go
func TestOrderRefund(t *testing.T) {
	tests := []struct {
		name  string
		steps []Action
		want  Order
	}{
		{"refund after partial ship refunds unshipped only", []Action{Place(3, 30), Ship(2), Refund()}, Order{Items: 3, Price: 30, Shipped: 2, Refunded: 10}},
		{"refund after full ship is refused", []Action{Place(3, 30), Ship(3), Refund()}, Order{Items: 3, Price: 30, Shipped: 3, Err: ErrAlreadyShipped}},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			got := Run(NewOrder(), tt.steps...)
			if !reflect.DeepEqual(got, tt.want) {
				t.Fatalf("got %+v, want %+v", got, tt.want)
			}
		})
	}
}
```

Why: the cases the user checked by clicking are now the cases the tests check, so the Go provably does what the prototype showed. Edge cases the port adds (input validation, errors from storage) get their own tests on top.

Then run code-review on the result.

## Don't
- Add tests to the prototype itself. Why: tests make it real code, and it is meant to be thrown away.
- Connect it to a real database or service. Use in-memory state unless the question is about persistence.
- Generalise for "what if we later need X". It answers one question.
- Ship the HTML page into the product. Only the answer, and for JS the pure module, moves on.
