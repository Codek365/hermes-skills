---
name: cmux-browser-4-hermes-ai
description: "Drive cmux browser surfaces: navigate, click, fill, inspect."
version: 0.1.0
author: Hermes
metadata:
  hermes:
    tags: [Browser, Cmux, Automation, CLI]
---
# cmux Browser for Hermes AI

Drive cmux browser surfaces (the cmux app's built-in browser) from Hermes via the `terminal` tool. Covers navigation, DOM interaction, inspection, JS eval, state, tabs, dialogs, frames, and downloads. Does NOT replace `browser_navigate`/`browser_click` — those target the Hermes browser stack; this skill targets the user's cmux browser surfaces that share the user's authenticated profiles. Requires the cmux CLI (`/Applications/cmux.app/Contents/Resources/bin/cmux` on macOS).

## When to Use

- User says "use cmux browser" or refers to cmux surfaces/panes.
- Need to automate a page that is already open in the user's cmux browser.
- Need authenticated browsing against the user's real profiles (cmux surfaces inherit login state).
- Hermes `browser_navigate` lands on a login wall but the user has the page open in cmux.
- User asks to fill a form, scrape a page, or capture a screenshot via cmux.

## Prerequisites

- cmux app installed and running on macOS.
- CLI available as `cmux` on PATH (verify: `cmux --version`).
- Invoke all commands through the `terminal` tool.

## How to Run

Identify the target surface, then issue `cmux browser <subcommand>` calls through the `terminal` tool. Most subcommands take a surface target positionally or via `--surface`.

## Quick Reference

Navigation: `open`, `open-split`, `navigate`, `back`, `forward`, `reload`, `url`, `identify`
Waiting: `wait` (--selector/--text/--url-contains/--load-state/--function/--timeout-ms)
DOM: `click`, `dblclick`, `hover`, `focus`, `check`, `uncheck`, `scroll-into-view`, `type`, `fill`, `press`, `keydown`, `keyup`, `select`, `scroll`
Inspection: `snapshot`, `screenshot`, `get`, `is`, `find`, `highlight`
JS: `eval`, `addinitscript`, `addscript`, `addstyle`
State: `cookies`, `storage`, `state`, `history`
Tabs: `tab list`, `tab new`, `tab switch`, `tab close`
Logs: `console list`, `console clear`, `errors list`, `errors clear`
Dialogs: `dialog accept`, `dialog dismiss`
Frames: `frame "<selector>"`, `frame main`
Downloads: `download --path <file> --timeout-ms <ms>`

## Procedure

1. Discover surfaces: invoke `cmux browser identify` through `terminal`.
   ```
   cmux browser identify
   ```
2. Open a page (creates a new split if needed):
   ```
   cmux browser open https://example.com
   ```
3. Wait for page readiness:
   ```
   cmux browser surface:2 wait --load-state complete --timeout-ms 15000
   cmux browser surface:2 wait --selector "#checkout" --timeout-ms 10000
   cmux browser surface:2 wait --text "Order confirmed"
   cmux browser surface:2 wait --url-contains "/dashboard"
   cmux browser surface:2 wait --function "window.__appReady === true"
   ```
4. Interact with DOM (mutating actions accept `--snapshot-after`):
   ```
   cmux browser surface:2 click "button[type='submit']" --snapshot-after
   cmux browser surface:2 fill "#email" --text "ops@example.com"
   cmux browser surface:2 fill "#email" --text ""        # clear
   cmux browser surface:2 type "#search" "cmux"
   cmux browser surface:2 press Enter
   cmux browser surface:2 select "#region" "us-east"
   cmux browser surface:2 scroll --dy 800 --snapshot-after
   cmux browser surface:2 scroll-into-view "#pricing"
   ```
5. Inspect page state:
   ```
   cmux browser surface:2 snapshot --interactive --compact
   cmux browser surface:2 screenshot --out /tmp/cmux-page.png
   cmux browser surface:2 get title
   cmux browser surface:2 get url
   cmux browser surface:2 get text "h1"
   cmux browser surface:2 get html "main"
   cmux browser surface:2 get value "#email"
   cmux browser surface:2 get attr "a.primary" --attr href
   cmux browser surface:2 get count ".row"
   cmux browser surface:2 get box "#checkout"
   cmux browser surface:2 get styles "#total" --property color
   cmux browser surface:2 is visible "#checkout"
   cmux browser surface:2 is enabled "button[type='submit']"
   cmux browser surface:2 is checked "#terms"
   cmux browser surface:2 find role button --name "Continue"
   cmux browser surface:2 find label "Email"
   cmux browser surface:2 find testid "save-btn"
   cmux browser surface:2 find first ".row"
   cmux browser surface:2 find nth 2 ".row"
   ```
6. Eval JavaScript or inject scripts/styles:
   ```
   cmux browser surface:2 eval "document.title"
   cmux browser surface:2 eval --script "window.location.href"
   cmux browser surface:2 addinitscript "window.__cmuxReady = true;"
   cmux browser surface:2 addscript "document.querySelector('#name')?.focus()"
   cmux browser surface:2 addstyle "#debug-banner { display: none !important; }"
   ```
7. Manage session state:
   ```
   cmux browser surface:2 cookies get
   cmux browser surface:2 cookies get --name session_id
   cmux browser surface:2 cookies set session_id abc123 --domain example.com --path /
   cmux browser surface:2 cookies clear --name session_id
   cmux browser surface:2 cookies clear --all
   cmux browser surface:2 storage local set theme dark
   cmux browser surface:2 storage local get theme
   cmux browser surface:2 storage local clear
   cmux browser surface:2 storage session set flow onboarding
   cmux browser surface:2 state save /tmp/cmux-browser-state.json
   cmux browser surface:2 state load /tmp/cmux-browser-state.json
   cmux browser surface:2 history clear --force
   ```
8. Manage tabs:
   ```
   cmux browser surface:2 tab list
   cmux browser surface:2 tab new https://example.com/pricing
   cmux browser surface:2 tab switch 1
   cmux browser surface:2 tab switch surface:7
   cmux browser surface:2 tab close
   cmux browser surface:2 tab close surface:7
   ```
9. Handle dialogs, frames, downloads:
   ```
   cmux browser surface:2 dialog accept
   cmux browser surface:2 dialog accept "Confirmed by automation"
   cmux browser surface:2 dialog dismiss
   cmux browser surface:2 frame "iframe[name='checkout']"
   cmux browser surface:2 click "#pay-now"
   cmux browser surface:2 frame main
   cmux browser surface:2 click "a#download-report"
   cmux browser surface:2 download --path /tmp/report.csv --timeout-ms 30000
   ```
10. Capture debug artifacts on failure:
    ```
    cmux browser surface:2 console list
    cmux browser surface:2 errors list
    cmux browser surface:2 screenshot --out /tmp/cmux-failure.png
    cmux browser surface:2 snapshot --interactive --compact
    ```

## Common Patterns

### Primary browser workspace for Google Workspace

When the user wants Hermes to avoid taking over their normal browser/desktop and the work is mostly browser-based, prefer cmux browser surfaces before `computer_use`. For Google Workspace, have the user log into the cmux browser normally, then reuse that authenticated surface; this is not a permission bypass and must not involve copying cookies/tokens. For multi-account Google sessions, use explicit `u/<index>` or `authuser=<index>` URLs and verify the active account in the snapshot before any mutating work. See `references/google-workspace-cmux-profile.md` for the tested ktobi@matina.io pattern and commands.

Navigate-wait-inspect:
```
cmux browser open https://example.com/login
cmux browser surface:2 wait --load-state complete --timeout-ms 15000
cmux browser surface:2 snapshot --interactive --compact
cmux browser surface:2 get title
```

Fill form and verify:
```
cmux browser surface:2 fill "#email" --text "ops@example.com"
cmux browser surface:2 fill "#password" --text "$PASSWORD"
cmux browser surface:2 click "button[type='submit']" --snapshot-after
cmux browser surface:2 wait --text "Welcome"
cmux browser surface:2 is visible "#dashboard"
```

Persist and restore session:
```
cmux browser surface:2 state save /tmp/session.json
# ...later...
cmux browser surface:2 state load /tmp/session.json
cmux browser surface:2 reload
```

## References

- `references/cmux-command-reference.md` — full condensed command reference from cmux.com/docs/browser-automation; consult for exact flags/syntax without re-fetching the web docs.
- `references/sandbox-routing.md` — decision matrix for combining cmux browser, Camofox persistent sandbox, dedicated Chrome CDP profile, and computer_use so Hermes can avoid stealing the user's desktop while still using authenticated browser sessions.
- `references/google-workspace-cmux-profile.md` — tested pattern for reusing a user-completed Google Workspace login in cmux, including `u/<index>` / `authuser=<index>` account selection and safety boundaries.
- `references/appscript-crud-via-cmux.md` — tested Apps Script CRUD pattern: REST API create/read/update with `script.projects`, plus cmux authenticated UI cleanup when Drive API trash/delete is blocked by Workspace policy.

## Blazor App Login via cmux (Task9 pattern)

Blazor Server apps (e.g. Task9 UI) use interactive forms where `fill`/`type` on `input[ref=eN]` selectors may fail because cmux ref selectors are not CSS. Use `eval` to set values and dispatch events:

```bash
# 1. Fill credentials via eval (works reliably with Blazor's input binding)
cmux browser surface:3 eval --script "
  const inputs = document.querySelectorAll('input');
  inputs[0].value = 'testAdmin';
  inputs[0].dispatchEvent(new Event('input', {bubbles:true}));
  inputs[1].value = 'password';
  inputs[1].dispatchEvent(new Event('input', {bubbles:true}));
  'filled'
"

# 2. Click login button — try form submit first, then button click
cmux browser surface:3 eval --script "
  const form = document.querySelector('form');
  if (form) { form.requestSubmit(); 'form submitted' }
  else {
    document.querySelectorAll('button').forEach(b => {
      if (b.textContent.includes('Đăng nhập') || b.type === 'submit') b.click()
    });
    'btn clicked'
  }
"

# 3. Wait + verify redirect (Blazor redirects after auth)
sleep 5 && cmux browser surface:3 get url
# Should show /report-seo-performance or /dashboard, not /login
```

Key points:
- Blazor input binding requires `Event('input', {bubbles:true})` dispatch after setting `.value` — just setting value without the event won't register.
- Login button may not have `type=submit`; try both `form.requestSubmit()` and `button.click()`.
- After login, session persists in cmux's browser profile — subsequent navigations within the same surface stay authenticated.
- Session can expire between cron runs; always check `get url` for redirect to `/login` before interacting with authenticated pages.

## Pitfalls

- Surface IDs (`surface:2`) are dynamic per session — always `identify` first; do not hardcode. If `cmux browser identify` returns `not_found: Surface not found or not a browser`, create/open a browser surface with `cmux browser open <url>` and parse the returned `OK surface=surface:N`; then target that surface explicitly for all subsequent commands.
- `type` appends text; use `fill` to replace the entire field value.
- For web tasks where the user wants Hermes not to steal the desktop or be affected by their windows, prefer routing in this order: cmux browser surface → Camofox persistent sandbox → dedicated Chrome CDP profile → `computer_use` only as a GUI/native fallback. See `references/sandbox-routing.md`.
- `fill --text ""` clears the field.
- `--snapshot-after` is available on mutating actions (click, fill, scroll, etc.) and returns a compact DOM snapshot immediately — use it to verify in one round-trip.
- A surface may be a webview embedded in cmux; `focus-webview` / `is-webview-focused` check whether the webview has keyboard focus.
- Downloads require a prior user-gesture (e.g. click) before calling `download`.
- Frame scoping: after `frame "<selector>"`, all subsequent commands run in that iframe until `frame main` resets to the top-level document.
- `addinitscript` runs on every navigation before page scripts; `addscript` runs once immediately.
- cmux browser surfaces share the user's authenticated browser profiles — do not use for anonymous/scraping tasks where isolation is needed; use `browser_navigate` instead.
- Do NOT print cookies, tokens, or storage values unless the user explicitly asks.
- Publishing to a GitHub skills repo with `hermes skills publish` when gh has multiple accounts: `gh auth switch --user <account>` may not persist for the publish subprocess. Set `GH_TOKEN=$(gh auth token --user <account>)` before calling `hermes skills publish` to ensure the correct token is used. The publish command creates a PR (not a direct push); merge it with `gh pr merge` afterwards.

## Verification

```
cmux browser identify
cmux browser surface:2 get title
```
Returns the active surface ID and page title, confirming the surface is reachable and ready for automation.