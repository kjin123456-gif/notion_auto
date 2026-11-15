# CLAUDE.md

Top-level project rules. Claude Code reads this file first in every session.

## 1. Project Purpose

- Register and look up schedules based on Notion databases.
- After user approval, perform follow-up actions (mail, Docs/Sheets, Drive) through the GWS CLI.

## 2. Work Approval Rule (Todo List First)

- For every task the user requests, **output a todo list first and report it before doing any work.**
- Each todo item states what will be done and which files, databases, or external services it touches.
- **Start work only after the user approves the todo list.** If the user requests changes, revise the list and report it again.
- If the scope changes during work, stop, report the updated todo list, and get approval again.

## 3. Input Classification Rules (Highest Priority)

1. **Priority 1 — `keyword:content` format**
   - Classify by trigger keyword → **registration path**.
2. **Priority 2 — natural language without a keyword**
   - Refer to `docs/intent-map.md` → **lookup / execution path**.
3. **Cannot decide**
   - Do not guess. Ask the user again.

## 4. Memo Classification Rules

When the user enters a memo, classify it by prefix using the table below and save it to Notion.

| Memo prefix | Rule to read first | Target DB |
|---|---|---|
| `개인:` or `personal:` | `.claude/rules/nt-personal.md` | Personal Tasks DB |
| `업무:`, `강의:`, or `work:` | `.claude/rules/nt-work.md` | Work DB |
| `배움:`, `공부:`, `학습:`, or `study:` | `.claude/rules/nt-study.md` | Study DB |

- After classifying by prefix, read the matching rule file first, then save to the DB according to that rule.
- When saving is done, output only one line: `저장됨:[DB name]` (e.g. `저장됨:Work DB`).
- This one-line output overrides the result report format (`templates/register-result.md`) for memo registrations.

## 5. Processing Flow by Path (Summary)

Details: `docs/workflow.md`

- **Registration path:** receive input → detect separator → match trigger keyword → parse content → register in target DB → report result (`templates/register-result.md`; for memos, the one-line output in section 4).
- **Lookup / execution path:** receive input → analyze intent (`docs/intent-map.md`) → query and analyze Notion → present report (`templates/report.md`) → write action plan (`workspace/plans/`, `templates/action-plan.md`) → user approval → `gws-executor` executes → report and log result (`workspace/logs/`).

## 6. Role Assignment

| Role | Responsibility |
|------|----------------|
| Main session | Input classification, reporting, todo lists, approval requests |
| `notion-agent` | Notion DB table creation, schedule classification and registration, filtering (date, status, assignee), and analysis reports |
| `gws-executor` | Executes **approved plans only** via GWS CLI |

## 7. Approval Principles

Details: `docs/approval-policy.md`

- External actions such as sending mail and creating or sharing files **must be approved before execution**.
- Mail defaults to **draft**. Sending requires separate, explicit approval.
- Do not change sharing settings unless explicitly approved.

## 8. Documentation Language and Translation Rules

- **All `.md` files must be written in English.**
- A Korean translation of every `.md` file is stored separately in the `translate/` folder.
- **Every translated `.md` file name must start with `_`.**
  - Example: `CLAUDE.md` → `translate/_CLAUDE.md`
  - Example: `docs/workflow.md` → `translate/docs/_workflow.md`
- Folder names are kept as-is and the original folder structure is mirrored under `translate/`.
  - Example: `.claude/` → `translate/.claude/` (e.g. `.claude/agents/notion-agent.md` → `translate/.claude/agents/_notion-agent.md`)
- Whenever an English `.md` file is created or modified, create or update its Korean translation in the same task.

## 9. Reference Documents

- `docs/`
  - `workflow.md` — full processing flow
  - `approval-policy.md` — approval rules
  - `intent-map.md` — natural-language intent mapping
  - `notion-setup.md` — Notion DB setup guide
  - `gws-commands.md` — GWS CLI command reference
- `templates/`
  - `register-result.md`, `report.md`, `action-plan.md`, `mail/reminder.md`
- `config/`
  - `notion.json` — Notion settings
  - `gws.json` — GWS settings (default folders, etc.)
- `.claude/`
  - `agents/` (`notion-agent`, `gws-executor`), `skills/` (`notion-db-register`, `notion-db-analyze`, `gws-execute`)
- `workspace/`
  - `plans/`, `logs/`, `outputs/`

## 10. Prohibited Actions

- Sending mail without approval.
- Deleting Notion items.
- Printing secret values (`.env`, tokens, API keys).
- Starting work before the todo list is approved.

## 11. Response Language and Report Format

- Respond to the user in **Korean**.
- Reports follow the formats in `templates/`, except the memo save output defined in section 4.
- Documentation files (`.md`) remain in English; Korean versions live only in `translate/`.
