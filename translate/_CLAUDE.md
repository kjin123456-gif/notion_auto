# CLAUDE.md (한글 번역본)

프로젝트 최상위 규칙 문서. Claude Code는 매 세션 이 파일을 가장 먼저 읽는다.

## 1. 프로젝트 목적

- 노션 DB 기반으로 일정을 등록·조회한다.
- 사용자 승인 후 GWS CLI로 후속 작업(메일, Docs/Sheets, Drive)을 수행한다.

## 2. 작업 승인 규칙 (Todo List 선보고)

- 사용자가 요청하는 모든 작업은 **작업 전에 todo list를 먼저 출력하여 보고한다.**
- 각 todo 항목에는 수행할 내용과 영향을 주는 파일·DB·외부 서비스를 명시한다.
- **사용자가 todo list를 승인한 후에만 작업을 시작한다.** 수정 요청 시 목록을 고쳐 다시 보고한다.
- 작업 중 범위가 바뀌면 중단하고, 변경된 todo list를 보고해 다시 승인받는다.

## 3. 입력 분류 규칙 (최우선)

1. **1순위 — `키워드:내용` 형식**
   - 트리거 키워드로 분류 → **등록 경로**
2. **2순위 — 키워드 없는 자연어**
   - `docs/intent-map.md` 참조 → **조회·실행 경로**
3. **판단 불가 시**
   - 추측하지 않고 사용자에게 재질문한다.

## 4. 경로별 처리 흐름 (요약)

상세: `docs/workflow.md`

- **등록 경로:** 입력 수신 → 구분자 판별 → 트리거 키워드 대조 → 내용 파싱 → 대상 DB 등록 → 결과 보고(`templates/register-result.md`)
- **조회·실행 경로:** 입력 수신 → 의도 분석(`docs/intent-map.md`) → 노션 조회·분석 → 보고서 제시(`templates/report.md`) → 실행 계획 작성(`workspace/plans/`, `templates/action-plan.md`) → 사용자 승인 → `gws-executor` 실행 → 결과 보고·기록(`workspace/logs/`)

## 5. 역할 분담

| 역할 | 담당 |
|------|------|
| 메인 세션 | 입력 분류, 보고, todo list, 승인 요청 |
| `notion-agent` | 노션 DB 표 생성, 일정 분류·등록, 필터링(날짜·상태·담당자), 분석 보고 |
| `gws-executor` | **승인된 계획만** GWS CLI로 실행 |

## 6. 승인 원칙

상세: `docs/approval-policy.md`

- 메일 발송, 파일 생성·공유 등 외부 작업은 **반드시 승인 후 실행**한다.
- 메일 기본값은 **초안**이며, 발송은 별도 명시 승인이 필요하다.
- 명시적 승인 없이 공유 설정을 변경하지 않는다.

## 7. 문서 언어 및 번역 규칙

- **모든 `.md` 파일은 영어로 작성한다.**
- 모든 `.md` 파일의 한글 번역본은 `translate/` 폴더에 따로 저장한다.
- **번역본 `.md` 파일 이름 앞에는 반드시 `_`를 붙인다.**
  - 예: `CLAUDE.md` → `translate/_CLAUDE.md`
  - 예: `docs/workflow.md` → `translate/docs/_workflow.md`
- 폴더 이름은 그대로 두고, 원본 폴더 구조를 `translate/` 아래에 동일하게 유지한다.
  - 예: `.claude/` → `translate/.claude/` (예: `.claude/agents/notion-agent.md` → `translate/.claude/agents/_notion-agent.md`)
- 영어 `.md` 파일을 생성·수정하면 같은 작업 안에서 한글 번역본도 생성·갱신한다.

## 8. 참조 문서 목록

- `docs/`
  - `workflow.md` — 전체 처리 흐름
  - `approval-policy.md` — 승인 규칙
  - `intent-map.md` — 자연어 의도 매핑
  - `notion-setup.md` — 노션 DB 설정 안내
  - `gws-commands.md` — GWS CLI 명령 참조
- `templates/`
  - `register-result.md`, `report.md`, `action-plan.md`, `mail/reminder.md`
- `config/`
  - `notion.json` — 노션 설정
  - `gws.json` — GWS 설정(기본 저장 폴더 등)
- `.claude/`
  - `agents/` (`notion-agent`, `gws-executor`), `skills/` (`notion-db-register`, `notion-db-analyze`, `gws-execute`)
- `workspace/`
  - `plans/`, `logs/`, `outputs/`

## 9. 금지 사항

- 승인 없는 메일 발송
- 노션 항목 삭제
- 비밀값 출력(`.env`, 토큰, API 키)
- todo list 승인 전 작업 시작

## 10. 응답 언어 및 보고 형식

- 사용자에게는 **한국어**로 응답한다.
- 보고는 `templates/` 양식을 따른다.
- 문서 파일(`.md`)은 영어로 유지하며, 한글본은 `translate/`에만 둔다.
