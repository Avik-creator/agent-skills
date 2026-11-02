# UI prototype

Read when the question is what a page or component should look like. You build several structurally different variants on one real route, switchable with `?variant=` and a floating bar, so the user can flip between them, pick one, or take parts from each.

## 1. Pick the host page
- **Default: an existing page.** Render the variants on the route that already exists, choosing one with `?variant=`. The page's data loading, params and auth stay as they are; only the rendered part changes. Something new that would naturally sit inside an existing page (a new card, a new section, a new step in a flow) still goes on that page. Why: next to the real header, data and density, design problems show up; on an empty page every variant looks fine.
- **Last resort: a new route.** Only when nothing existing could host it. Follow the project's routing convention, and put `prototype` in the path, such as `/prototype/<name>`.

## 2. Show the plan, then build
Show the user one line and build only after a yes:

> 3 variants of the settings page on `/settings?variant=`: A a single long form, B a sidebar of sections, C a search-first command list.

Why: three variants is a lot of code to throw away if the route or the question is wrong. Use 3 variants by default and never more than 5, because past that they stop being really different.

Write the plan in `NOTES.md`, and record the host page as a file you touched.

## 3. Make the variants really different
- Each variant differs in layout, information hierarchy, or the main thing the user does. Colour or copy changes don't count. If two drafts come out alike, redo one with an explicit ban, such as "no card grid".
- Use the project's own component library and styling.
- One file per variant next to the page, named `<Page>.variant-a.prototype.tsx` (or the project's extension), each exporting a component with a short name, such as `VariantB` named "Sidebar sections".
- Share small pieces (a header) at most, never a shared layout. Why: each variant must be free to throw the layout out.
- Writes go to a stub that logs and returns success. Reads use the page's existing local or dev data, never production. Sample data is made up.

## 4. Wire them into the page
Keep the page's data loading above the switch, and change only what renders:

```tsx
const variant = searchParams.get("variant") ?? "A";
return (
  <>
    {variant === "A" && <VariantA {...data} />}
    {variant === "B" && <VariantB {...data} />}
    {variant === "C" && <VariantC {...data} />}
    <PrototypeSwitcher variants={["A", "B", "C"]} current={variant} />
  </>
);
```

- Before editing the host page, check it has no uncommitted changes (`git status --short <page>`). If it has, ask first. Why: undoing the wiring later must not throw away the user's own work.
- Write down exactly what you changed in the host page in `NOTES.md`, so it can be undone by hand.

## 5. The floating switcher
One small component, named `PrototypeSwitcher.prototype.tsx`, in the folder where the project keeps shared UI:
- A fixed pill at the bottom centre: a left arrow, the current key and name (such as "B · Sidebar sections"), and a right arrow. The arrows wrap around.
- Clicking an arrow updates `?variant=` through the framework's router (`router.replace`, `navigate`), so a variant can be reloaded and shared.
- The ← and → keys also switch, except when an `<input>`, `<textarea>` or `[contenteditable]` has focus.
- It looks clearly separate from the design, such as a dark high-contrast pill with a shadow.
- It renders nothing in production builds (`process.env.NODE_ENV === "production"` or the project's equivalent). Why: a stray merge must not show it to users.

## 6. Check every variant, then hand it over
- Start the dev server with the project's normal command, and open each `?variant=` key with the automation skill (browser-use first). Screenshot each one and check it renders with data, and that the arrows and keys switch. Why: the user shouldn't open a blank variant.
- Give the user the URL and the keys. The useful feedback is usually "the header from B with the sidebar from C", which is the real design.

## 7. Capture the answer and clean up
- Follow SKILL.md "Capture the answer": which variant won, and why.
- Undo the host page wiring using `NOTES.md`, then delete the variant files and the switcher (or keep them on the local branch first).

## 8. Move the winner into real code
Rewrite the winner properly in the page itself. The variant was built without tests or error handling.
1. Write the tests first for what the user will see and do: the states it shows (loading, empty, error, full), and the main actions. Watch them fail.
2. Build the winner into the page with real data and real writes.
3. Run code-review on the result.

## Don't
- Make variants that differ only in colour or copy.
- Connect variants to real writes.
- Promote a variant file straight into the product.
