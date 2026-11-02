# browser-use

Read when using browser-use (step 1): the user's own Chrome, or a Browser Use Cloud browser when no local Chrome is usable.

## Check it first

```bash
browser-use --doctor      # is it installed, and can it reach Chrome
browser-use skill show    # the usage guide for the installed version
```

When `skill show` disagrees with this file, follow `skill show`. Why: it ships with the installed version, and this file doesn't.

The current CLI runs Python from stdin. The old `open`, `state`, `click`, `input`, `tab`, `cloud connect` and `--connect` commands are removed; don't use them.

## Running commands

Helpers are pre-imported. Keep multi-step work in one heredoc when practical.

```bash
browser-use <<'PY'
new_tab("http://localhost:3000/login")
wait_for_load()
print(page_info())
PY
```

| Need | Helpers |
|---|---|
| Navigate | `new_tab(url)`, `goto_url(url)`, `wait_for_load()` |
| Inspect | `page_info()`, `js(code)`, `cdp(method, ...)` |
| Interact | `click_at_xy(x, y)`, `type_text(text)`, `fill_input(selector, text)`, `press_key(key)`, `scroll(x, y)` |
| Tabs | `list_tabs()`, `current_tab()`, `switch_tab(target)`, `close_tab(target)` |
| Proof | `capture_screenshot()`, `wait_for_element(selector)` |

## In the user's Chrome

- The first navigation of a task is `new_tab(url)`. After that, reuse the same tab. Don't open a new tab in every command.
- Only touch tabs you opened. Close them when the task is done.
- Never call `activate_tab()` unless the user asks to see the tab. Why: it brings Chrome to the front over whatever the user is doing. Screenshots and input work in the background.
- Use the default daemon. Don't set a new `BU_NAME` per task. Why: each new daemon makes Chrome ask for permission again.
- Remote debugging must be on at `chrome://inspect/#remote-debugging`. The tool opens that page if it's off.
- On macOS, Chrome may show "Allow remote debugging?". Leave the waiting command running and run `browser-use mac-approve` in a second call. Don't rerun the original command. If it reports `accessibility-required`, ask the user once to grant Accessibility access to the app running the agent (System Settings → Privacy & Security → Accessibility), then run `mac-approve` again.
- If the daemon goes stale, let it recover, or run `--doctor`. A timeout or a closed tab is not a reason to start another daemon.

## Finding and clicking elements

Use the accessibility tree, not screenshots. Why: it names every element and costs far fewer tokens than an image.

```python
nodes = cdp("Accessibility.getFullAXTree")["nodes"]   # thousands of nodes: filter before printing
button = next(n for n in nodes if n.get("name", {}).get("value") == "Sign in")
q = cdp("DOM.getBoxModel", backendNodeId=button["backendDOMNodeId"])["model"]["content"]
click_at_xy(sum(q[0::2]) / 4, sum(q[1::2]) / 4)       # box centre, in viewport pixels
wait_for_load()
print(page_info())                                    # confirm the click did what you expected
```

- Use a targeted `js(...)` query only when the tree lacks the element (canvas, unusual widgets).
- Take a screenshot when layout or images matter, or to record a failure.
- Check every action: `page_info()`, a focused DOM check, or a screenshot.

## Cloud browser

Use it only when there is no usable local Chrome (a server, a sandbox) or for tasks that must run in parallel. Ask first. Why: it bills until it stops; the README lists $0.02 per browser-hour (checked 2026-11-02).

```bash
browser-use auth login              # once; check with: browser-use auth status

browser-use <<'PY'
start_remote_daemon("r7k2")         # a short unique name, one per parallel task
PY

BU_NAME=r7k2 browser-use <<'PY'
new_tab("https://example.com")
wait_for_load()
print(page_info())
PY

browser-use <<'PY'
stop_remote_daemon("r7k2")          # always stop it when done
PY
```

Use the same `BU_NAME` for every command in that browser. Don't start a cloud browser and then keep using the local one.
