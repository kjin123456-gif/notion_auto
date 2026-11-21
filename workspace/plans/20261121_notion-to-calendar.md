# Action Plan: Notion schedules → Google Calendar

## 1. Plan Info

| Item | Value |
|------|-------|
| Plan ID | 20261121_notion-to-calendar |
| Created | 2026-11-21 16:20 (KST) |
| Status | **executed** |
| Executed | 2026-11-21 16:22–16:30 (KST) by `gws-executor` |

## 2. Source Report Summary

- Source: Personal Tasks / Work / Study databases under the Notion page "개인 일정 관리" (connected to the `notion_auto` integration).
- Read-only scrape of the table views (Notion MCP was unavailable to `notion-agent`). Raw data: `workspace/outputs/notion-calendar-source.json`.
- 10 items in total: Personal 1, Work 8, Study 1. 1 item has no date and is skipped.

## 3. Tasks

Target: the user's **primary** Google Calendar. Time zone: `Asia/Seoul`.
Rules: date-only → all-day event; date with time → 1-hour event starting at that time.
Each event gets the Notion page link in its description and `extendedProperties.private.notionPageId` for duplicate checks.
No attendees, no invitations, no sharing changes.

| # | DB | Color (colorId) | Title | Start | End | Status |
|---|----|-----------------|-------|-------|-----|--------|
| 1 | Personal | Flamingo pink (4) | 남양주 방문 (location: 남양주) | 2026-11-21 (all day) | — | done |
| 2 | Work | Tomato red (11) | 정규수업 개강 | 2026-11-17 (all day) | — | done |
| 3 | Work | Tomato red (11) | 주차별 학습목표·차시 구성안 (XLSX) [김민준/박지현] | 2026-11-18 18:00 | 19:00 | done |
| 4 | Work | Tomato red (11) | 강의 슬라이드 초안 제작 (PPTX) [이서연/김민준] | 2026-11-20 18:00 | 19:00 | done |
| 5 | Work | Tomato red (11) | 실습과제·학습자 안내자료 (DOCX→PDF) [박지현/이서연] | 2026-11-23 15:00 | 16:00 | done |
| 6 | Work | Tomato red (11) | 커리큘럼 다음 점검회의 (온라인) [전원] | 2026-11-24 16:00 | 17:00 | done |
| 7 | Work | Tomato red (11) | 통합 검수 및 수정의견 취합 (XLSX) [최도윤/전원] | 2026-11-24 17:00 | 18:00 | done |
| 8 | Work | Tomato red (11) | 최종본 확정 및 공유 (PPTX, PDF, XLSX) [김민준/전원] | 2026-11-25 18:00 | 19:00 | done |
| 9 | Study | Basil green (10) | 에이전트 업데이트 사항 업데이트하기 | 2026-11-18 (all day) | — | done |

Skipped: Work "수정된 커리큘럼 표 전달" (no 마감일).

## 4. GWS Commands

For each task, first check for an existing event, then insert:

```
gws calendar events list --params '{"calendarId":"primary","privateExtendedProperty":"notionPageId=<PAGE_ID>"}'
gws calendar events insert --params '{"calendarId":"primary"}' --json '{"summary":"<title>","colorId":"<id>","start":{...},"end":{...},"description":"Notion: <url>","extendedProperties":{"private":{"notionPageId":"<PAGE_ID>"}}}'
```

All-day: `{"date":"YYYY-MM-DD"}` start, next day as end. Timed: `{"dateTime":"...+09:00","timeZone":"Asia/Seoul"}`.

## 5. Expected Result

- 9 new events in the primary calendar, colored pink / red / green by database.
- Re-running the plan creates no duplicates.
- Log: `workspace/logs/20261121_notion-to-calendar.log`.

## 6. Approval

| Item | Value |
|------|-------|
| Approved by | user (main session) |
| Approved at | 2026-11-21 16:23 (KST) |
| Method | chat reply "승인" |

## 7. Execution Results

All duplicate checks returned no existing events; all 9 inserts succeeded.

| # | Event ID |
|---|----------|
| 1 | 8v9aoc2taoqo2ndhjpvrqiseig |
| 2 | da9364a8r4mfrig8nohv0j9n50 |
| 3 | k5n309ilku16iefp2cm89jg1cc |
| 4 | 9kqs2n4lt707rmmke37di8abao |
| 5 | vavqa6rg77tmg3sa4c3675dgv4 |
| 6 | 1ebgir6720f7jr8sir8etsmroo |
| 7 | vqm3cjq491lsh6goug0u8tu7a0 |
| 8 | cif4hncea852cqe42nkthoflco |
| 9 | 8kdfq41vrpfc8sqpcgc5l1dpgg |

Details and links: `workspace/logs/20261121_notion-to-calendar.log`.
