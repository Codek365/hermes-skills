# Google Workspace authenticated cmux browser profile pattern

Use this reference when the user wants Hermes to work in a browser sandbox/workspace without taking over their normal desktop browser, especially for Google Workspace / Apps Script / Sheets tasks.

## What was verified

- cmux CLI: `/Applications/cmux.app/Contents/Resources/bin/cmux`.
- `cmux browser open https://example.com` can create a browser surface and returns `OK surface=surface:N ...` even when `cmux browser identify` says no browser surface is focused.
- After the user logs into Google inside the cmux browser, the cmux browser surface can reuse that authenticated session for Google Workspace.
- Verified Google account button text in snapshots can show the active account, e.g. `Google Account: K Tobi (ktobi@matina.io)`.
- Verified Google Workspace pages accessible through cmux after login:
  - `https://docs.google.com/spreadsheets/u/3/` → Google Sheets
  - `https://myaccount.google.com/u/3/` / `https://myaccount.google.com/?authuser=3` → Google Account
  - `https://script.google.com/u/3/home` / `https://script.google.com/home?authuser=3` → Apps Script project list
  - `https://drive.google.com/drive/u/3/my-drive` → Google Drive

## Safe boundary

Do not extract, print, copy, or inject cookies, tokens, localStorage, sessionStorage, or Chrome profile secrets. This workflow is session reuse after the user logs in normally, not a login/permission bypass. If Google asks for password, 2FA, consent, or admin approval, stop and ask the user to complete it.

## Recommended workflow

```bash
CMUX="/Applications/cmux.app/Contents/Resources/bin/cmux"

# 1. Try to discover a focused browser surface.
$CMUX browser identify

# 2. If identify fails with `Surface not found or not a browser`, create/open one.
OUT=$($CMUX browser open 'https://docs.google.com/spreadsheets/u/3/' 2>&1)
echo "$OUT"
# Parse `surface:9` from: OK surface=surface:9 pane=... placement=...

# 3. Target the explicit surface for all later calls.
S=surface:9
$CMUX browser "$S" wait --load-state complete --timeout-ms 20000
$CMUX browser "$S" get url
$CMUX browser "$S" get title
$CMUX browser "$S" snapshot --interactive --compact
```

## Google multi-account pitfall

Google can keep multiple accounts in one browser session. When working as a specific Workspace account, include the `u/<index>` or `authuser=<index>` segment that matches the logged-in account. For the verified ktobi@matina.io session, `u/3` / `authuser=3` selected the right account.

Useful URLs:

```text
https://docs.google.com/spreadsheets/u/3/
https://myaccount.google.com/?authuser=3
https://script.google.com/u/3/home
https://script.google.com/home?authuser=3
https://drive.google.com/drive/u/3/my-drive
```

Always verify the active account in a snapshot before doing mutating work:

```bash
$CMUX browser "$S" snapshot --interactive --compact | grep -i 'Google Account'
```

Do not assume `u/3` is universal across machines or sessions. If the snapshot shows the wrong account or a login page, ask the user to switch/login in cmux, then retry.

## Command quirks learned

- `text=Learn more` is not a valid selector for `cmux browser click`; use CSS selectors like `a`, role/name discovery (`find role button --name ...`), or JS eval when necessary.
- For form tests, `fill`, `check`, and `click` worked, and `--snapshot-after` gave immediate verification.
- `cmux browser navigate https://script.google.com/home` may appear to stay on a previous Google app if account selection is ambiguous; prefer explicit `https://script.google.com/u/3/home` or `?authuser=3` and verify URL/title afterward.
