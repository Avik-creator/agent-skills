# Writing unit and integration tests

Read before writing a unit or integration test (SKILL.md step 4). Examples are TypeScript with Vitest; the rules hold in any runner. Go tests follow the go skill's `references/testing.md`.

## Shape of a test
- **Arrange, act, assert**, with a blank line between them. One behaviour per test; several `expect`s are fine when they check the same behaviour.
- **Name** it after the behaviour, in the repo's style. If the repo has none: `when <situation>, then <result>`.
- **Flat**: no `if`, loops, `try/catch` or logging in the test body. Why: a test with branches can pass down the branch that checks nothing.
- **Only what affects the result** is in the test. Anything that does belongs in the test or its `beforeEach`, never hidden in another file.
- **Typed**: no `any`. Feed an invalid input with `value as unknown as Order`, so the cast is visible.
- **No assertions in helpers or hooks**. Why: the failure should point at the test.

```typescript
test('when filtering by active, then only active orders are shown', async () => {
  const active = buildOrder({ status: 'active', customerName: 'Ada Lovelace' })
  const completed = buildOrder({ status: 'completed', customerName: 'Alan Turing' })
  server.use(http.get('/api/orders', () => HttpResponse.json([active, completed])))
  render(<OrdersReport />)

  await userEvent.click(screen.getByRole('button', { name: 'Active only' }))

  expect(screen.getByRole('cell', { name: active.customerName })).toBeVisible()
  expect(screen.queryByRole('cell', { name: completed.customerName })).toBeNull()
})
```

What this does right: the asserted names come from the setup; it also checks the order that must not appear; it mocks only the HTTP boundary; it finds elements as a user does.

```typescript
it('should test orders filtering', async () => {                        // vague name
  const admin = { role: 'admin' }                                       // most privileged role hides permission bugs
  const svc = vi.fn()                                                   // mocks the app's own code
  render(<OrdersReport data={[{ id: 1, name: 'test1' }]} />)            // meaningless data
  try {                                                                  // a branch in the test
    await userEvent.click(screen.getByTestId('filter'))                  // a test id the repo doesn't use
    expect(screen.getAllByRole('row').length).toBe(5)                    // 5 never appears in the setup
    expect(svc).toHaveBeenCalled()                                       // checks a call, not a result
  } catch (e) { console.log(e) }                                         // swallows the failure
})
```

## Many inputs, one behaviour
Use the runner's table form for the edge bank rows (`test.each`, `@pytest.mark.parametrize`, a Go table). One row per class, each row named after its case, so a failure says which edge broke.

```typescript
test.each([
  { name: 'empty', input: '', valid: false },
  { name: 'only spaces', input: '   ', valid: false },
  { name: 'at the limit', input: 'a'.repeat(64), valid: true },
  { name: 'one past the limit', input: 'a'.repeat(65), valid: false },
])('when the name is $name, then valid is $valid', ({ input, valid }) => {
  expect(isValidName(input)).toBe(valid)
})
```

## Assertions
- Compare the whole value (`toEqual`) when you can. One full comparison shows the whole diff and replaces several weak checks.
- Every value in an assertion appears in the setup: `expect(result.id).toBe(order.id)`, never `toBe('123')`.
- Errors: assert the type and the context, never a bare "it threw". `await expect(getUser(id)).rejects.toThrow(NotFoundError)`.
- Async: await every assertion. Where the runner supports it, add `expect.hasAssertions()` so a skipped assertion fails the test.
- Go a little further: test a save with two items, a filter with an item that must not appear, a boolean flag in both states with each set explicitly. Pick the least-privileged role that should succeed.

## Test data
- Use meaningful values a reader recognises ("Ada Lovelace", a real-looking amount), not `test1`.
- Use a build function with defaults and overrides (`buildOrder({ status: 'active' })`) once the third test needs the same object. Override only the fields the test is about. Why: before that, inline data is easier to read.
- Random data only with a fixed seed. Why: a red test must fail the same way twice.
- Database rows: unique fields get a unique suffix; each test creates the rows it acts on.
- When a builder default could hide a bug (a flag that defaults to true), set it explicitly in the test, or name the default in a one-line comment on the builder.

## Test doubles

| Double | What it is | Use for |
|---|---|---|
| Fake | A small working version (an in-memory store) | A stateful dependency you can't run |
| Stub | Returns canned data | Controlling what a dependency returns |
| Spy | Wraps the real thing and records calls | Checking a call happened when the call is the behaviour (an email sent) |
| Mock | Replaces it and records calls | Rarely; only when both are needed |

- Double only the external edge: network, third-party APIs, email, payments, the clock. The app's own modules, router and state run for real. Why: a suite where every collaborator is mocked stays green while the wiring is broken.
- For HTTP, intercept the network (MSW, nock, `httptest` in Go) rather than mocking the client function, and make an unhandled request fail the test. Check the option name in the version the repo uses; some default to only a warning.
- Stub data is copied from a real response, typed with the real response type, never `as any` or `as never`. One line above it says where it came from. Destructure the result the way the real caller does, so a renamed field fails.
- Reset doubles, environment variables and globals in `beforeEach`.

## Integration tests
- Run the real router, handlers and database. Use the same database engine the project uses, started locally. Why: another engine passes queries the real one rejects.
- Seed only shared reference data (countries, currencies). Each test creates its own records.
- Assert through the public API (the HTTP response, the next read), not by querying tables, unless the stored row is the behaviour.
- Add several records and assert only the intended ones changed. Test cascading deletes and updates when the change touches them.
- A rollback test uses a real transaction against the real database.
- Auto-generated fields: `expect.any(Number)` rather than a guessed id.

## Fake timers
- Turn them on per test and back off in `afterEach`. Fake only what the test needs.
- Start the promise, advance time, then await:
  ```typescript
  const pending = fetchWithTimeout(url)
  await vi.advanceTimersByTimeAsync(5_000)
  await expect(pending).rejects.toThrow(TimeoutError)
  ```
  Awaiting first hangs, because time is frozen.
- Don't use them for plain `async` code with no timers.

## Snapshots
Only for structured output that is tedious to assert field by field (a rendered email, CLI output). Keep them small and reviewed. Use property matchers for ids and timestamps. In CI, run so that a missing snapshot fails instead of being written (Jest: `--ci`).
