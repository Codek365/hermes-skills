# Apps Script CRUD via API + cmux authenticated UI fallback

Use this when the user wants to manage Google Apps Script projects from Hermes while logged into Google Workspace in cmux. This pattern was tested with ktobi@matina.io in cmux and an OAuth token that has `script.projects` scope.

## Safety boundary

- Reuse only an authenticated session the user completed in cmux or an OAuth token the user already granted.
- Do not copy, print, or transplant cookies, localStorage, access tokens, refresh tokens, or session storage.
- This is not a permission bypass. If Google or Workspace policy blocks an action, report the block and use a normal UI/API path only when the account is authorized.

## Tested result

Apps Script REST API worked for code CRUD:

- Create project: `POST https://script.googleapis.com/v1/projects` returned 200 with a `scriptId`.
- Read content: `GET https://script.googleapis.com/v1/projects/{scriptId}/content` returned 200.
- Update content: `PUT https://script.googleapis.com/v1/projects/{scriptId}/content` returned 200 and persisted `Code.gs` plus `appsscript.json`.
- Verify update: a second GET confirmed the new function source existed.

Drive API delete/trash did not work in the tested Workspace:

```text
HTTP 403
The domain administrators have disabled Drive apps.
reason: domainPolicy
```

cmux UI cleanup did work:

- Navigate to `https://script.google.com/u/3/home`.
- Find the project in My Projects.
- Open the row's More Options menu.
- Choose `Remove`.
- Confirm `REMOVE`.
- Verify the project no longer appears in the My Projects list.

After UI remove, `projects/{scriptId}/content` may still be readable. Treat UI remove as removing/trashing from Apps Script home, not proof of permanent deletion.

## Account selection

For multi-account Google sessions, force the intended account before Apps Script work:

```bash
cmux browser surface:N navigate 'https://myaccount.google.com/?authuser=3'
cmux browser surface:N wait --load-state complete --timeout-ms 20000
cmux browser surface:N snapshot --interactive --compact
```

Completion criterion: the snapshot shows the intended account label, for example:

```text
Google Account: K Tobi (ktobi@matina.io)
```

Use `/u/3/` or `authuser=3` URLs for Apps Script/Drive/Sheets after that:

```bash
cmux browser surface:N navigate 'https://script.google.com/u/3/home'
cmux browser surface:N navigate 'https://drive.google.com/drive/u/3/my-drive'
cmux browser surface:N navigate 'https://docs.google.com/spreadsheets/u/3/'
```

## API smoke test shape

Run API calls through the `terminal` tool using the existing OAuth token file. Do not print token values.

```python
import json, urllib.request
from pathlib import Path

tok = json.loads((Path.home()/'.hermes/google_token.json').read_text())
access = tok['token']
headers = {'Authorization': 'Bearer ' + access, 'Content-Type': 'application/json'}

# Create
body = {'title': 'Hermes CRUD Smoke Test'}
req = urllib.request.Request(
    'https://script.googleapis.com/v1/projects',
    data=json.dumps(body).encode(), method='POST', headers=headers)
created = json.loads(urllib.request.urlopen(req, timeout=30).read())
script_id = created['scriptId']

# Read
req = urllib.request.Request(
    f'https://script.googleapis.com/v1/projects/{script_id}/content',
    method='GET', headers={'Authorization': 'Bearer ' + access})
content = json.loads(urllib.request.urlopen(req, timeout=30).read())

# Update: PUT replaces the entire project content, so include every file.
files = [
  {'name': 'appsscript', 'type': 'JSON', 'source': json.dumps({
    'timeZone': 'Asia/Ho_Chi_Minh',
    'exceptionLogging': 'STACKDRIVER',
    'runtimeVersion': 'V8'
  }, indent=2)},
  {'name': 'Code', 'type': 'SERVER_JS', 'source': "function hermesCrudSmokeTest() {\n  return 'ok-' + new Date().toISOString();\n}\n"}
]
req = urllib.request.Request(
    f'https://script.googleapis.com/v1/projects/{script_id}/content',
    data=json.dumps({'files': files}).encode(), method='PUT', headers=headers)
updated = json.loads(urllib.request.urlopen(req, timeout=30).read())
```

Verification criteria:

- Create returns a `scriptId`.
- Read returns at least `appsscript` JSON.
- Update returns both `appsscript` and `Code` files.
- A second read contains `hermesCrudSmokeTest` in the `Code` source.

## cmux UI remove recipe

Use this only for a test project or when the user explicitly asks to remove a project.

```bash
CMUX=/Applications/cmux.app/Contents/Resources/bin/cmux
S=surface:N
$CMUX browser "$S" navigate 'https://script.google.com/u/3/home'
$CMUX browser "$S" wait --load-state complete --timeout-ms 20000
$CMUX browser "$S" snapshot --interactive --compact
```

Open the row menu for the specific project. Prefer exact project title matching via JS so you do not remove the wrong project:

```bash
$CMUX browser "$S" eval --script "(() => {
  const title = 'Hermes CRUD Smoke Test';
  const rows = [...document.querySelectorAll('[role=option], [role=row], div')];
  const row = rows.find(e => (e.innerText || e.textContent || '').includes(title));
  if (!row) return 'project-not-found';
  const btn = row.querySelector('[role=button][aria-label="More Options"]') ||
              row.querySelector('[aria-label*="More"]');
  if (!btn) return 'more-options-not-found';
  btn.click();
  return 'opened-menu';
})()"
```

Then inspect the menu and choose Remove only when the menu belongs to the intended row:

```bash
$CMUX browser "$S" snapshot --interactive --compact
$CMUX browser "$S" eval --script "(() => {
  const item = [...document.querySelectorAll('[role=menuitem]')]
    .find(e => (e.getAttribute('aria-label') || e.innerText || '').includes('Remove') &&
               e.getAttribute('aria-disabled') !== 'true');
  if (!item) return 'remove-not-found';
  item.click();
  return 'clicked-remove';
})()"
```

Confirm only after the dialog appears:

```bash
$CMUX browser "$S" snapshot --interactive --compact
$CMUX browser "$S" eval --script "(() => {
  const btn = [...document.querySelectorAll('div[role=button], button')]
    .find(e => (e.innerText || e.textContent || '').trim() === 'REMOVE');
  if (!btn) return 'confirm-remove-not-found';
  btn.click();
  return 'confirmed-remove';
})()"
```

Completion criterion: a follow-up snapshot/search no longer contains the project title in My Projects.

## Pitfalls

- `cmux browser identify` can return `not_found` even though a created surface still works by explicit ID. If `cmux browser open` returns `OK surface=surface:N`, target that surface explicitly.
- Google may keep the same page title while routing between Apps Script and Sheets if a single-page app fails to swap state. Verify URL and snapshot, not title alone.
- `script.projects` is enough for Apps Script content create/read/update, but Drive trash/delete can still be blocked by Workspace policy.
- `PUT /content` replaces the whole Apps Script project. Always read first and include all existing files when updating a real project.
- Avoid permanent `Delete forever` during smoke tests. UI `Remove` is enough to verify cleanup capability without irreversible deletion.
