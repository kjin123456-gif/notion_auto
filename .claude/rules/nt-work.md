# nt-work

Trigger rule for work-related to-dos. It decides which inputs are registered in the Notion **Work** database.

## Priority

This rule takes precedence over any other instruction file (`CLAUDE.md`, skills, agents, `docs/`) on how trigger keywords are classified. If they conflict, follow this rule.

## Trigger Keywords

- `업무`, `강의`, `work`
- Input format: `<keyword>:<content>` (e.g. `업무:...`)

## Target Database

- Notion **Work** database (the database ID is set in `config/notion.json`).

## What Belongs Here

Meetings, reports and deliverables, deadlines, client or colleague requests, business trips, work mail.

Examples:

- `업무:주간 보고서 제출 금요일 5시`
- `work:client meeting Tuesday 2pm`
- `업무:견적서 검토 내일까지`

## Input Without a Keyword

- If the user clearly asks to add a to-do and the content fits this category, classify it here and use the registration path (`notion-db-register`).
- If the content could also fit `nt-personal` or `nt-study`, do not guess. Ask the user which category it belongs to.
- If the input is a lookup or analysis request rather than a new to-do, use the lookup path (`notion-db-analyze`).

## Exceptions

- If a keyword from this rule is used but the content clearly belongs to another category, register it here as the keyword says, and mention the possible mismatch in the result report.
