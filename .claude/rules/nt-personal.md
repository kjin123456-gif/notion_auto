# nt-personal

Trigger rule for personal to-dos. It decides which inputs are registered in the Notion **Personal Tasks** database.

## Priority

This rule takes precedence over any other instruction file (`CLAUDE.md`, skills, agents, `docs/`) on how trigger keywords are classified. If they conflict, follow this rule.

## Trigger Keywords

- `개인`, `personal`
- Input format: `<keyword>:<content>` (e.g. `개인:...`)

## Target Database

- Notion **Personal Tasks** database (the database ID is set in `config/notion.json`).

## What Belongs Here

Errands, appointments with friends or family, health and hospital visits, hobbies, household chores, personal shopping.

Examples:

- `개인:친구와 카페약속 토요일 3시`
- `personal:dentist appointment Friday 10am`
- `개인:관리비 납부 25일까지`

## Input Without a Keyword

- If the user clearly asks to add a to-do and the content fits this category, classify it here and use the registration path (`notion-db-register`).
- If the content could also fit `nt-work` or `nt-study`, do not guess. Ask the user which category it belongs to.
- If the input is a lookup or analysis request rather than a new to-do, use the lookup path (`notion-db-analyze`).

## Exceptions

- If a keyword from this rule is used but the content clearly belongs to another category, register it here as the keyword says, and mention the possible mismatch in the result report.
