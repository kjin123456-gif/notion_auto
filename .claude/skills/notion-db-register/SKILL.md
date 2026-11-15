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

Split the text after the separator into the target database's properties. Property names and types are defined in `config/notion.json`.

| Database | Property | How to extract | Example |
|----------|----------|----------------|---------|
| Personal Tasks | `일정이름` (title) | Main phrase without the date, time, person, or place | `Cafe with a friend` |
| | `날짜` (date) | Relative or absolute expression → ISO 8601 (based on today's date) | `Saturday 3pm` → `2026-11-21T15:00` |
| | `사람` (person) | Notion user named in the input | `me` |
| | `중요도` (select) | `높음` / `중간` / `낮음` if stated | `높음` |
| | `위치` (text) | Place name in the input | `Gangnam station` |
| Work | `업무이름` (title) | Main phrase without the date, people, or mail address | `Submit weekly report` |
| | `등록날짜` (date) | Always today (auto-filled) | `2026-11-15` |
| | `마감일` (date) | Deadline or scheduled date in the input → ISO 8601 | `Friday 5pm` → `2026-11-20T17:00` |
| | `공동작업자` (person) | Notion users named in the input | — |
| | `전송메일` (email) | Mail address in the input | `someone@example.com` |
| | `중요도` (select) | `높음` / `중간` / `낮음` if stated | — |
| Study | `학습 주제` (title) | Main phrase without the date | `Python lecture 3` |
| | `마감일` (date) | Deadline or scheduled date → ISO 8601 | `Thursday` → `2026-11-19` |
| | `요약` (text) | Extra details in the input | — |

- Leave a property empty when it is not in the input. The databases have no status property; never add one.
- If the date or time is ambiguous (e.g. "next week" without a day), **do not guess**. Ask the user again.

### 3. Register through `notion-agent`

Call the `notion-agent` subagent with:

```json
{
  "path": "register",
  "keyword": "<trigger keyword>",
  "target_db": "<target database>",
  "content": { "<property name>": "<value>", "...": "..." }
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
