# nt-study

Trigger rule for study-related to-dos. It decides which inputs are registered in the Notion **Study** database.

## Priority

This rule takes precedence over any other instruction file (`CLAUDE.md`, skills, agents, `docs/`) on how trigger keywords are classified. If they conflict, follow this rule.

## Trigger Keywords

- `학습`, `공부`, `study`
- Input format: `<keyword>:<content>` (e.g. `학습:...`)

## Target Database

- Notion **Study** database (the database ID is set in `config/notion.json`).

## What Belongs Here

Classes and lectures, assignments, exams, reading, online courses, certification preparation, practice and review.

Examples:

- `학습:파이썬 강의 3강 수강 오늘 밤`
- `study:exam review Thursday`
- `공부:영어 단어 50개 암기 매일`

## Input Without a Keyword

- If the user clearly asks to add a to-do and the content fits this category, classify it here and use the registration path (`notion-db-register`).
- If the content could also fit `nt-personal` or `nt-work`, do not guess. Ask the user which category it belongs to.
- If the input is a lookup or analysis request rather than a new to-do, use the lookup path (`notion-db-analyze`).

## Exceptions

- If a keyword from this rule is used but the content clearly belongs to another category, register it here as the keyword says, and mention the possible mismatch in the result report.
