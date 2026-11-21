# 실행 계획: Notion 일정 → Google Calendar

## 1. 계획 정보

| 항목 | 값 |
|------|-----|
| 계획 ID | 20261121_notion-to-calendar |
| 작성 일시 | 2026-11-21 16:20 (KST) |
| 상태 | **executed (실행 완료)** |
| 실행 일시 | 2026-11-21 16:22–16:30 (KST), `gws-executor` |

## 2. 근거 보고서 요약

- 출처: Notion 「개인 일정 관리」 페이지(`notion_auto` 통합 연결) 아래의 Personal Tasks / Work / Study DB.
- 표 보기 화면을 읽기 전용으로 수집했습니다(`notion-agent`에서 Notion MCP를 쓸 수 없었음). 원본 데이터: `workspace/outputs/notion-calendar-source.json`.
- 총 10건: Personal 1건, Work 8건, Study 1건. 날짜가 없는 1건은 건너뜁니다.

## 3. 작업 목록

대상: 사용자의 **기본(primary)** Google 캘린더. 시간대: `Asia/Seoul`.
규칙: 날짜만 있으면 종일 일정, 시간이 있으면 그 시각부터 1시간짜리 일정.
각 일정 설명에 Notion 페이지 링크를 넣고, 중복 확인용으로 `extendedProperties.private.notionPageId`를 넣습니다.
참석자 추가, 초대 발송, 공유 설정 변경은 하지 않습니다.

| # | DB | 색상 (colorId) | 제목 | 시작 | 종료 | 상태 |
|---|----|----------------|------|------|------|------|
| 1 | Personal | 분홍 Flamingo (4) | 남양주 방문 (위치: 남양주) | 2026-11-21 (종일) | — | 완료 |
| 2 | Work | 빨강 Tomato (11) | 정규수업 개강 | 2026-11-17 (종일) | — | 완료 |
| 3 | Work | 빨강 Tomato (11) | 주차별 학습목표·차시 구성안 (XLSX) [김민준/박지현] | 2026-11-18 18:00 | 19:00 | 완료 |
| 4 | Work | 빨강 Tomato (11) | 강의 슬라이드 초안 제작 (PPTX) [이서연/김민준] | 2026-11-20 18:00 | 19:00 | 완료 |
| 5 | Work | 빨강 Tomato (11) | 실습과제·학습자 안내자료 (DOCX→PDF) [박지현/이서연] | 2026-11-23 15:00 | 16:00 | 완료 |
| 6 | Work | 빨강 Tomato (11) | 커리큘럼 다음 점검회의 (온라인) [전원] | 2026-11-24 16:00 | 17:00 | 완료 |
| 7 | Work | 빨강 Tomato (11) | 통합 검수 및 수정의견 취합 (XLSX) [최도윤/전원] | 2026-11-24 17:00 | 18:00 | 완료 |
| 8 | Work | 빨강 Tomato (11) | 최종본 확정 및 공유 (PPTX, PDF, XLSX) [김민준/전원] | 2026-11-25 18:00 | 19:00 | 완료 |
| 9 | Study | 초록 Basil (10) | 에이전트 업데이트 사항 업데이트하기 | 2026-11-18 (종일) | — | 완료 |

건너뜀: Work 「수정된 커리큘럼 표 전달」 (마감일 없음).

## 4. GWS 명령

각 작업마다 먼저 기존 일정이 있는지 확인한 뒤 등록합니다.

```
gws calendar events list --params '{"calendarId":"primary","privateExtendedProperty":"notionPageId=<PAGE_ID>"}'
gws calendar events insert --params '{"calendarId":"primary"}' --json '{"summary":"<title>","colorId":"<id>","start":{...},"end":{...},"description":"Notion: <url>","extendedProperties":{"private":{"notionPageId":"<PAGE_ID>"}}}'
```

종일 일정: 시작은 `{"date":"YYYY-MM-DD"}`, 종료는 다음 날. 시간 일정: `{"dateTime":"...+09:00","timeZone":"Asia/Seoul"}`.

## 5. 예상 결과

- 기본 캘린더에 새 일정 9건이 생기고, DB에 따라 분홍 / 빨강 / 초록으로 표시됩니다.
- 계획을 다시 실행해도 중복으로 만들지 않습니다.
- 로그: `workspace/logs/20261121_notion-to-calendar.log`.

## 6. 승인 정보

| 항목 | 값 |
|------|-----|
| 승인자 | 사용자 (메인 세션) |
| 승인 일시 | 2026-11-21 16:23 (KST) |
| 승인 방식 | 채팅 답변 "승인" |

## 7. 실행 결과

중복 확인 결과 기존 일정은 없었고, 9건 모두 정상 등록되었습니다.

| # | 일정 ID |
|---|---------|
| 1 | 8v9aoc2taoqo2ndhjpvrqiseig |
| 2 | da9364a8r4mfrig8nohv0j9n50 |
| 3 | k5n309ilku16iefp2cm89jg1cc |
| 4 | 9kqs2n4lt707rmmke37di8abao |
| 5 | vavqa6rg77tmg3sa4c3675dgv4 |
| 6 | 1ebgir6720f7jr8sir8etsmroo |
| 7 | vqm3cjq491lsh6goug0u8tu7a0 |
| 8 | cif4hncea852cqe42nkthoflco |
| 9 | 8kdfq41vrpfc8sqpcgc5l1dpgg |

상세 내용과 링크: `workspace/logs/20261121_notion-to-calendar.log`.
