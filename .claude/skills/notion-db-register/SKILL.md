---
name: notion-db-register
description: Registers a schedule item in a Notion database when the user's input is in "keyword:content" format and the keyword matches a trigger. Use it after the input has been classified as the registration path (e.g. "personal:cafe with a friend Saturday 3pm"). It splits the content into the target database's properties, registers it through the notion-agent subagent, and hands any follow-up action to the gws-execute skill.
---

# notion-db-register

Registers a trigger-classified item in its target Notion database.

## When to Use

- The input contains a separator (`:`) and the part before it matches a trigger keyword.
- Do **not** use it for natural-language requests without a keyword; use `notion-db-analyze` instead.

## Procedure

### 1. Decide the target database

- Match the trigger keyword → decide the target database.

### 2. Split the content into properties

Split the text after the separator into the target database's properties:

| Property | How to extract | Example |
|----------|----------------|---------|
| Title | Main phrase without the date, time, or assignee | `Cafe with a friend` |
| Date / time | Relative or absolute expressions converted to ISO 8601 (based on today's date) | `Saturday 3pm` → `2026-11-21T15:00` |
| Status | Explicit status word if present, otherwise the default | `Not started` |
| Assignee | Name or "me" if present, otherwise the default | `me` |
| Other | Any extra property the target DB has (tags, priority, etc.) | — |

- Leave a property empty when it is not in the input; `notion-agent` applies the default values.
- If the date or time is ambiguous (e.g. "next week" without a day), **do not guess**. Ask the user again.

### 3. Register through `notion-agent`

Call the `notion-agent` subagent with:

```json
{
  "path": "register",
  "keyword": "<trigger keyword>",
  "target_db": "<target database>",
  "content": { "title": "...", "date": "...", "status": "...", "assignee": "..." }
}
```

### 4. Report the result

- Report the result to the user in Korean, using the `templates/register-result.md` format.

### 5. Hand off follow-up actions

- If the input or the user asks for a follow-up action (e.g. a reminder mail, a Docs/Sheets file), hand it to the **`gws-execute`** skill together with the registration result.
- Never run GWS actions directly from this skill.

## Exceptions

| Situation | Action |
|-----------|--------|
| Keyword does not match any trigger | Do not register. Show the available keywords and ask again |
| Date or time cannot be parsed | Do not register. Ask the user for the date or time |
| Property mismatch reported by `notion-agent` | Report the mismatched properties and ask how to proceed |
| Database inaccessible | Report the cause and stop |
