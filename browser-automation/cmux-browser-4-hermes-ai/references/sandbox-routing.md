# cmux + sandbox browser routing

Use this when the user wants Hermes browser work to avoid stealing their visible desktop, avoid interference from their own windows, or operate with a separate authenticated browser workspace.

## Default routing order

1. **cmux browser surface** — use when the user explicitly says cmux, the target page is already open in cmux, or the task needs the user's cmux-authenticated profile.
2. **Camofox persistent sandbox** — use when the user wants an isolated browser with persistent login/cookies and optional VNC/noVNC live view.
3. **Dedicated Chrome CDP profile** — use when the site requires real Chrome, but still avoid the user's main Chrome profile.
4. **computer_use** — use only for native macOS apps or GUI tasks that cannot be done via DOM/browser automation. Scope to a specific app and avoid raising windows unless explicitly requested.

## cmux surface workflow

```bash
CMUX="/Applications/cmux.app/Contents/Resources/bin/cmux"
$CMUX browser identify
$CMUX browser open https://example.com
$CMUX browser surface:2 wait --load-state complete --timeout-ms 15000
$CMUX browser surface:2 snapshot --interactive --compact
```

If `identify` returns `Surface not found or not a browser`, do not conclude cmux is broken. It usually means no browser surface is active/reachable. Ask the user to open/create a cmux browser surface, or create one with `cmux browser open <url>` when appropriate, then run `identify` again.

## Camofox persistent browser sandbox

Best for a separate agent browser profile with cookies/logins persisted across runs and no interference with the user's desktop browser.

Key config:

```bash
hermes config set CAMOFOX_URL http://localhost:9377
```

```yaml
browser:
  camofox:
    managed_persistence: true
```

When running Camofox in Docker with VNC/noVNC, expose the control and view ports, persist the server profile directory, and let the user login once inside the sandbox. noVNC is commonly available at `http://localhost:6080` when enabled.

## Dedicated Chrome CDP profile

Use a separate user-data-dir; do not attach Hermes to the user's main Chrome profile.

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.hermes/chrome-agent" \
  --no-first-run \
  --no-default-browser-check
```

Then connect from Hermes CLI with `/browser connect`.

## Privacy and safety

- Do not print cookies, tokens, localStorage, or session state unless explicitly requested.
- Prefer separate agent profiles over the user's primary Chrome/Safari profile.
- Treat cmux as authenticated/non-anonymous; use sandbox browser tools for anonymous or isolated browsing.
- computer_use is the fallback, not the default, for web tasks.