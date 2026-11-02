# Writing browser and end-to-end tests

Read before writing or fixing a browser test. Examples use Playwright with TypeScript; follow the runner the repo already has. Checking a UI by hand is the automation skill; call it before writing a browser test, as SKILL.md says.

## Scope
- Main user flows only: sign-up, checkout, the flow the feature exists for. Edge cases and validation rules go in lower-level tests. Why: end-to-end tests are slow and point at the cause least well.
- Don't mock the app's own backend. Mock only third parties you can't run, and assert that the app handed off to them (the URL changed, the request was sent), never their pages.

## Shape of a test
- **Arrange** fresh data through the API or database, not by clicking through the UI. **Act** through the user's journey. **Assert** what the user sees and what was stored.
- **Name** it after who does what and what they get: `a customer can buy an item and sees the order confirmation`.
- One journey per test. Use `test.step` to label the parts of a long journey, so the report says which step failed.
- Each test stands alone: it passes alone, in any order, and in parallel.

```typescript
test('a customer can buy an item and sees the order confirmation', async ({ page, api }) => {
  const customer = await api.createCustomer({ name: 'Ada Lovelace' })
  const product = await api.createProduct({ name: 'Desk lamp', price: 4_500 })
  await loginAs(page, customer)

  await page.goto(`/products/${product.id}`)
  await page.getByRole('button', { name: 'Add to cart' }).click()
  await page.getByRole('link', { name: 'Checkout' }).click()
  await page.getByRole('button', { name: 'Place order' }).click()

  await expect(page).toHaveURL(/\/orders\/\d+$/)
  await expect(page.getByRole('heading', { name: 'Order confirmed' })).toBeVisible()
  await expect(page.getByRole('row', { name: /Desk lamp/ })).toContainText('45.00')
})
```

What this does right: its own data, made through the API; it finds elements as a user does; it asserts what the user sees (URL, heading, the line on the order) rather than internal state.

```typescript
test('Should purchase item', async ({ page }) => {                     // vague name
  await page.goto('/products')                                          // assumes a logged-in session
  await page.getByText('iPhone 15 Pro').click()                         // assumes data that may not exist
  await page.locator('#add-to-cart-btn').click()                        // CSS id
  await page.waitForTimeout(3000)                                       // a sleep
  await page.locator('button').last().click()                           // position, not meaning
  expect(await page.locator('.cart-item').isVisible()).toBe(true)       // a one-time read, no retry
  expect(await page.evaluate(() => localStorage.orderId)).toBeTruthy()  // internal state
})
```

## Helpers and page objects
- Write the steps inline first. When the same steps appear in a third test, extract a helper named after the user's step (`loginAs`, `addToCart`, `placeOrder`), not after the click (`clickButton`). Why: a step helper reads like the journey; a click helper hides it.
- Use a page object only when several tests drive the same page and its locators repeat. It holds locators and actions and returns values; it never asserts. Prefer a small object per page over a base-class hierarchy.
- Data setup helpers that call the API (`api.createCustomer`) are fine from the first test, because they keep the UI out of the arrange step. Use a fixture to hand them to the test.

## Config that matters
Check the repo's config has these before adding tests. Ask before changing it, since the whole suite uses it.
- `baseURL`, and a `webServer` that starts the app locally.
- `trace: 'retain-on-failure'` and `screenshot: 'only-on-failure'`.
- `forbidOnly` on in CI, so a stray `test.only` fails the run.
- Retries: ask the user. If they are on, also run with `--fail-on-flaky-tests` so a pass after a retry still fails.
- A setup project that logs in once (see "Data and login").

## Finding elements
- Find elements the way a user does: role and accessible name, label, visible text. Use test ids only where the repo already uses them.
- Matching is a substring by default (`getByLabel('Password')` also matches "Confirm password"). Pass `exact: true` when another element could match.
- No CSS chains, XPath or `nth()` unless the order itself is what you are testing. Narrow inside a region instead: `page.getByRole('row', { name: /Desk lamp/ }).getByRole('button', { name: 'Remove' })`.

## Waiting
- Use assertions that retry until they pass (`await expect(locator).toHaveText(...)`). Never a sleep.
- One-time reads don't retry: `isVisible()`, `textContent()`, `allTextContents()`, `locator.all()`, `evaluate()`. Don't assert on them directly. Assert the count before looping over a list.
- Don't wait for `networkidle`; Playwright marks it discouraged. Wait for what the user would see.
- Set up `page.route` or `waitForResponse` before the action that triggers the request.
- No `force: true`, no try/catch around a timeout, no "if it's visible, click it". Control the state instead. Why: each one hides a real failure.

## Trusting a pass
- Before a check, assert you are on the right page: the URL and an element only the ready page has. Why: a login, blank or error page passes many checks, including accessibility scans.
- Absence checks (`toBeHidden`, `toHaveCount(0)`) pass when the locator is wrong. Prove the locator finds the element in the state where it is present.
- To prove a save, reload and read it back before anything else writes. Test removals and empty states as well as additions.

## Data and login
- A fresh browser context does not isolate server data. A test that changes an account, cart or setting needs its own records.
- Log in once in a setup project and reuse the saved `storageState`. Put the auth folder in `.gitignore`, because it holds session cookies. Tests that change server-side state get one account per worker.
- Credentials come from environment variables for a local or test account, never real ones.

## Failure paths
- Make the backend fail and assert what the user sees: `route.fulfill({ status: 500 })`, `route.abort('timedout')`, `context.setOffline(true)`. A stuck spinner or a dead click is a failing test.

## Accessibility a tool can decide
- After Escape, focus returns to the control that opened the dialog. `aria-expanded` flips when a menu opens and closes. After a failed submit, focus moves to the first error, which has `aria-invalid` and is described by its message.
- An axe scan of the ready page, including open dialogs and menus. The WCAG tags add up: list `wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa` as well as `wcag22aa`. A clean scan is not conformance.

## On failure
Read the trace before changing the test. A red end-to-end test is an app bug until the trace shows otherwise; see flaky.md.
