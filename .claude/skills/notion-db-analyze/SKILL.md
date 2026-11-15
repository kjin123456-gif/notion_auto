---
name: notion-db-analyze
description: Handles natural-language lookup and analysis requests that contain no trigger keyword (e.g. "check today's deadlines", "show this week's in-progress work for me"). It works out the period, conditions, and target databases from docs/intent-map.md, queries and filters them through the notion-agent subagent, reports the analysis, and proposes follow-up actions that can be handed to the gws-execute skill.
---

# notion-db-analyze

Processes lookup and analysis requests written in natural language without a keyword.

## When to Use

- The input has **no** `keyword:content` format, and the user wants to look up, filter, or analyze schedules or tasks.
- If the input has a trigger keyword, use `notion-db-register` instead.

## Procedure

### 1. Interpret the request

Using `docs/intent-map.md`, decide:

| Item | Example |
|------|---------|
| Period | today, this week, a date range |
| Conditions | importance (`중요도`: `높음`, `중간`, `낮음`), person (`사람` / `공동작업자`), other properties |
| Target databases | one or more databases from the intent map |
| Analysis type | deadline check, importance summary, per-person summary, etc. |

- If the intent is unclear or matches several meanings, **do not guess**. Ask the user again.

### 2. Query and analyze through `notion-agent`

Call the `notion-agent` subagent with:

```json
{
  "path": "query",
  "conditions": { "period": "...", "importance": "...", "person": "..." }
}
```

`notion-agent` sorts by nearest deadline and returns counts by importance, overdue items, items due soon, and items per person.

### 3. Report

- Report to the user in Korean, using the `templates/report.md` format.
- If there are no results, say so and show the conditions that were used.

### 4. Propose follow-up actions

- Suggest useful follow-up actions based on the result (e.g. a reminder mail draft, a summary sheet).
- If the user wants one, hand it to the **`gws-execute`** skill together with the report.
- Never run GWS actions directly from this skill.

## Exceptions

| Situation | Action |
|-----------|--------|
| Intent unclear | Ask the user again with likely options |
| No matching items | Report "no matching items" with the conditions used |
| Database inaccessible | Report the cause and stop |
