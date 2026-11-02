# agent-desktop

Read when using desktop control (step 3), after the user has said yes: native apps, or when neither browser tool works.

## Set up

```bash
npm install -g agent-desktop          # the user runs this, not you
agent-desktop version
agent-desktop permissions --guide     # macOS: grants Accessibility permission
```

- **macOS** needs Accessibility permission.
- **Windows** needs no grant for apps running at the terminal's privilege level.
- **Linux** needs the user's graphical session with `at-spi2-core`.

## Load its guide first

```bash
agent-desktop skills get desktop      # the loop, references, verification and retry rules
agent-desktop skills get platform     # what differs on this operating system
agent-desktop <command> --help        # flags for one command
```

Follow these over this file. Why: they ship with the installed version.

## The loop

Take a snapshot, act on a reference, then take a snapshot again to check:

```bash
agent-desktop list-apps
agent-desktop snapshot --app Finder -i      # interactive elements, each with a reference
agent-desktop click @s8f3k2p9:e8            # a reference from that snapshot
agent-desktop snapshot --app Finder -i      # did it do what you expected?
```

- A reference belongs to its snapshot (`@<snapshot>:eN`). A bare `@e8` is rejected.
- `STALE_REF` or `AMBIGUOUS_TARGET` means the screen changed: take a new snapshot and use its references.
- With several windows open, `snapshot` lists the window ids. Pass one with `--window-id`.
- For a busy app, map it first (`snapshot --skeleton --app Slack -i --compact`), then open one region (`snapshot --root <ref> -i --compact`).
- Every command prints JSON with `ok`, and either `data` or `error`. Retry only when `error.disposition.retry` is `safe`.

## The mouse and keyboard

By default it acts through the accessibility system, without moving the pointer or taking focus, so the user can keep working. The `--headed` flag allows real mouse and keyboard input. Ask before using it. Why: it takes over the user's mouse and keyboard while it runs.

## Privacy

Snapshots, screenshots, clipboard contents and debug files can hold whatever is on the user's screen: messages, documents, passwords.

- Read the clipboard only when the task needs it.
- Keep that output out of public logs, issues and uploads.
- Don't use its optional JEV helper without asking. Why: it sends screen descriptions to an outside model provider.
