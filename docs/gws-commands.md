# gws-commands

Reference for the GWS CLI (`gws`, Google Workspace CLI) commands used in this project. `gws-executor` and the `gws-execute` skill build commands from this document.

- Tool: `@googleworkspace/cli` (npm), verified on `gws 0.22.5`
- Authorized scopes: Gmail (`gmail.modify`), Drive, Docs, Sheets
- Command form: `gws <service> <resource> [sub-resource] <method> [flags]`
- Helper commands start with `+` (e.g. `gws gmail +send`).

## 1. Install, Authentication, and Version

| Command | Purpose |
|---|---|
| `npm install -g @googleworkspace/cli` | Install (Node.js 18+) |
| `gws --version` | Check the installed version |
| `gws auth status` | Show account, scopes, and token validity |
| `gws auth login -s gmail,drive,docs,sheets` | Log in with the scopes this project needs |
| `gws auth setup --project <id>` | Configure a GCP project and OAuth client (requires `gcloud`) |
| `gws auth logout` | Clear saved credentials |
| `gws schema <service.resource.method>` | Show the parameters of an API method |

- Credentials are stored encrypted in `%USERPROFILE%\.config\gws\credentials.enc`; the key is kept in the OS keyring.
- The OAuth client file is `%USERPROFILE%\.config\gws\client_secret.json`. Never commit it or print it.
- The OAuth app is in **Testing** mode, so the refresh token expires after **7 days**. On an auth error (exit code 2), run `gws auth login -s gmail,drive,docs,sheets` again.
- Do not run `gws auth export`: it prints decrypted secrets.

## 2. Gmail

| Command | Purpose | Key options |
|---|---|---|
| `gws gmail +triage` | Summarize unread mail (sender, subject, date) | `--max <N>`, `--query "<gmail query>"`, `--labels` |
| `gws gmail +read` | Read one message | `--id <MSG_ID>`, `--headers`, `--html` |
| `gws gmail +send ... --draft` | **Create a draft (default for this project)** | `--to`, `--cc`, `--bcc`, `--subject`, `--body`, `--html` |
| `gws gmail +send` (no `--draft`) | Send immediately. **Requires separate, explicit approval** | same as above |
| `gws gmail +reply` / `+reply-all` | Reply with threading handled automatically | `--message-id <ID>`, `--body`, `--draft` |
| `gws gmail +forward` | Forward a message | `--message-id <ID>`, `--to` |
| `gws gmail users drafts list` | List drafts | `--params '{"userId":"me"}'` |
| `gws gmail users drafts send` | Send an existing draft. **Requires explicit approval** | `--json '{"id":"<DRAFT_ID>"}'` |
| `gws gmail users messages list` | List messages (raw API) | `--params '{"userId":"me","q":"is:unread"}'` |

Example (reminder draft):

```bash
gws gmail +send --to "someone@example.com" --subject "Reminder: weekly report" \
  --body "The weekly report is due Friday 5pm." --draft
```

## 3. Drive

| Command | Purpose | Key options |
|---|---|---|
| `gws drive +upload <file>` | Upload a file with automatic metadata | `--parent <FOLDER_ID>` (from `config/gws.json`), `--name` |
| `gws drive files list` | Search and list files | `--params '{"q":"name contains '\''report'\''","pageSize":10}'` |
| `gws drive files get` | Get file metadata | `--params '{"fileId":"<ID>"}'` |
| `gws drive files create` | Create a folder or file | `--json '{"name":"<name>","mimeType":"application/vnd.google-apps.folder"}'` |
| `gws drive files copy` / `update` | Copy, rename, or move a file | `--params '{"fileId":"<ID>"}'`, `--json` |
| `gws drive files export` | Export a Docs/Sheets file (e.g. to PDF) | `--params '{"fileId":"<ID>","mimeType":"application/pdf"}' --output <path>` |

Example (upload a report to the default folder):

```bash
gws drive +upload workspace/outputs/report.md --parent "<FOLDER_ID>"
```

## 4. Docs

| Command | Purpose | Key options |
|---|---|---|
| `gws docs documents create` | Create a document | `--json '{"title":"<title>"}'` |
| `gws docs documents get` | Read a document | `--params '{"documentId":"<ID>"}'` |
| `gws docs +write` | Append plain text to the end of a document | `--document <ID> --text "<text>"` |
| `gws docs documents batchUpdate` | Insert, replace, or format content | `--params '{"documentId":"<ID>"}' --json '{"requests":[...]}'` |

New documents are created in the Drive root. Move them to the target folder with `gws drive files update` (`addParents`) if needed.

## 5. Sheets

| Command | Purpose | Key options |
|---|---|---|
| `gws sheets spreadsheets create` | Create a spreadsheet | `--json '{"properties":{"title":"<title>"}}'` |
| `gws sheets +read` | Read a range | `--spreadsheet <ID> --range 'Sheet1!A1:D10'` |
| `gws sheets +append` | Append rows | `--values "a,b,c"` or `--json-values '[["a","b"],["c","d"]]'` |
| `gws sheets spreadsheets values update` | Overwrite cell values | `--params '{"spreadsheetId":"<ID>","range":"...","valueInputOption":"USER_ENTERED"}' --json '{"values":[[...]]}'` |

Wrap ranges in single quotes: `!` triggers history expansion in bash.

## 6. Common Options and Output Format

| Option | Purpose |
|---|---|
| `--params <JSON>` | URL/query parameters |
| `--json <JSON>` | Request body (POST/PATCH/PUT) |
| `--upload <PATH>` / `--upload-content-type <MIME>` | Upload media content |
| `--output <PATH>` | Save a binary response to a file |
| `--format json\|table\|yaml\|csv` | Output format (default: `json`) |
| `--page-all`, `--page-limit <N>`, `--page-delay <MS>` | Auto-pagination (NDJSON, default limit 10 pages) |
| `--dry-run` | Show the request without executing it (helper commands) |

Exit codes:

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | API error |
| 2 | Auth error (credentials missing or invalid) |
| 3 | Validation error (bad arguments) |
| 4 | Discovery error (API schema could not be fetched) |
| 5 | Internal error |

Shell quoting: in Git Bash, use the examples as written. In Windows PowerShell 5.1, double quotes inside JSON may need escaping (`'{\"pageSize\": 5}'`).

## 7. Prohibited and Restricted Commands

| Command | Rule |
|---|---|
| `gws gmail +send` without `--draft`, `gws gmail users drafts send` | Only after separate, explicit approval (`docs/approval-policy.md`) |
| `gws drive files delete`, `gws drive files emptyTrash` | Prohibited (permanent deletion) |
| `gws gmail users drafts delete`, `gws gmail users messages delete` / `trash` | Prohibited unless the user explicitly asks |
| `gws drive permissions create` / `update` / `delete` | Do not change sharing settings unless explicitly approved |
| `gws auth export` | Prohibited (prints secrets) |

Run any approved write command with `--dry-run` first when the helper supports it, and include the preview in the action plan.

## 8. Common Errors and Fixes

| Symptom | Cause | Fix |
|---|---|---|
| Exit code 2 / `invalid_grant` | Token expired (7-day limit in Testing mode) or revoked | `gws auth login -s gmail,drive,docs,sheets` |
| `403 access_denied` in the browser | Account is not a test user of the OAuth app | Add the account under Google Auth Platform → Audience → Test users |
| `accessNotConfigured` (403) | API not enabled in the GCP project | `gcloud services enable <api>.googleapis.com --project <id>`, wait ~10 seconds, retry |
| `insufficient scopes` (403) | Scope not granted at login | Log in again with the missing scope (`-s ...,<service>`) |
| `redirect_uri_mismatch` | OAuth client is not a Desktop app | Create a new OAuth client of type Desktop app |
| Exit code 3 | Invalid JSON or arguments | Check quoting; verify parameters with `gws schema <method>` |
| Exit code 4 | API schema could not be fetched | Check the network and retry |
