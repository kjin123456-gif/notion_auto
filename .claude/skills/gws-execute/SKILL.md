---
name: gws-execute
description: Runs approved follow-up actions (Gmail drafts or sends, Drive uploads, Docs/Sheets creation) with the GWS CLI. Use it when notion-db-register or notion-db-analyze hands off a follow-up action, or when the user asks for a Google Workspace action. It checks the task type, reads the matching document in references/ if one exists, builds the commands, shows a preview, and runs them through the gws-executor subagent only after the user approves, then reports the result links.
---

# gws-execute

Turns a follow-up action into GWS CLI commands, gets approval, and runs it through `gws-executor`.

## When to Use

- A follow-up action is handed off from `notion-db-register` or `notion-db-analyze`.
- The user directly asks for a Gmail, Drive, Docs, or Sheets action.

## Reference Documents

- Path: `.claude/skills/gws-execute/references/`
- One document per task type (e.g. `gmail.md`, `drive.md`, `docs.md`, `sheets.md`).
- General command reference: `docs/gws-commands.md`.

## Procedure

### 1. Check the task type

Classify each requested action:

| Type | Examples |
|------|----------|
| Gmail | create a draft (default), send (only with separate explicit approval) |
| Drive | upload a file, put it in a folder (`config/gws.json`) |
| Docs | create a document and write its content |
| Sheets | create a sheet and write its content |

### 2. Read references and build commands

1. If `references/<type>.md` exists, read it before building the command.
2. If it does not exist, use `docs/gws-commands.md`.
3. Build the full `gws` command for each task. Fill in recipients, titles, folders, and content from the handed-off result and templates (`templates/mail/` for mail).
4. If a required value (recipient, folder, file name) is missing, **do not guess**. Ask the user.

### 3. Show a preview and ask for approval

1. Write an action plan in `workspace/plans/` using `templates/action-plan.md`, with status `pending`.
2. Show the user (in Korean):
   - The task list
   - **The exact commands to run** (secrets masked)
   - A preview of the content (mail body, document content)
   - For mail: draft or send
3. Wait for the user's answer.
   - Approved → set the plan status to `approved` and record the approver and approval time.
   - Changes requested → revise the plan and show the preview again.
   - Rejected → set the status to `rejected` and stop.

### 4. Run through `gws-executor`

- Only after approval, call the `gws-executor` subagent with the plan file path.
- Never run `gws` commands directly from this skill.

### 5. Report the result

- Report the result of each task to the user in Korean: **result links** (draft, file, document, sheet), IDs, and any failures.
- Mention where the log (`workspace/logs/`) and file copies (`workspace/outputs/`) are stored.

## Restrictions

- Never run anything before the user approves the preview.
- Mail is a **draft** by default. Sending needs separate, explicit approval.
- Never delete files or change sharing settings.

## Exceptions

| Situation | Action |
|-----------|--------|
| Required value missing | Ask the user before writing the plan |
| User rejects the plan | Mark it `rejected` and stop |
| `gws-executor` reports an authentication error | Report it with the re-authentication steps |
| `gws-executor` reports a failed command | Report the failed task. Do not retry automatically |
