# gws-commands

이 프로젝트에서 사용하는 GWS CLI(`gws`, Google Workspace CLI) 명령 레퍼런스. `gws-executor`와 `gws-execute` 스킬은 이 문서를 기준으로 명령을 구성한다.

- 도구: `@googleworkspace/cli` (npm), `gws 0.22.5`에서 확인
- 인증된 범위: Gmail(`gmail.modify`), Drive, Docs, Sheets
- 명령 형식: `gws <service> <resource> [sub-resource] <method> [flags]`
- 헬퍼 명령은 `+`로 시작한다 (예: `gws gmail +send`).

## 1. 설치, 인증, 버전 확인

| 명령 | 용도 |
|---|---|
| `npm install -g @googleworkspace/cli` | 설치 (Node.js 18 이상) |
| `gws --version` | 설치된 버전 확인 |
| `gws auth status` | 계정, 범위, 토큰 유효성 확인 |
| `gws auth login -s gmail,drive,docs,sheets` | 이 프로젝트에 필요한 범위로 로그인 |
| `gws auth setup --project <id>` | GCP 프로젝트와 OAuth 클라이언트 구성 (`gcloud` 필요) |
| `gws auth logout` | 저장된 인증 정보 삭제 |
| `gws schema <service.resource.method>` | API 메서드의 파라미터 확인 |

- 인증 정보는 `%USERPROFILE%\.config\gws\credentials.enc`에 암호화되어 저장되고, 키는 OS 키링에 보관된다.
- OAuth 클라이언트 파일은 `%USERPROFILE%\.config\gws\client_secret.json`이다. 커밋하거나 출력하지 않는다.
- OAuth 앱이 **테스트** 모드라서 refresh token은 **7일 후 만료**된다. 인증 오류(종료 코드 2)가 나면 `gws auth login -s gmail,drive,docs,sheets`를 다시 실행한다.
- `gws auth export`는 실행하지 않는다. 복호화된 비밀 값을 출력한다.

## 2. Gmail

| 명령 | 용도 | 주요 옵션 |
|---|---|---|
| `gws gmail +triage` | 읽지 않은 메일 요약 (보낸 사람, 제목, 날짜) | `--max <N>`, `--query "<gmail 검색어>"`, `--labels` |
| `gws gmail +read` | 메일 한 통 읽기 | `--id <MSG_ID>`, `--headers`, `--html` |
| `gws gmail +send ... --draft` | **초안 작성 (이 프로젝트의 기본값)** | `--to`, `--cc`, `--bcc`, `--subject`, `--body`, `--html` |
| `gws gmail +send` (`--draft` 없음) | 즉시 발송. **별도의 명시적 승인 필요** | 위와 같음 |
| `gws gmail +reply` / `+reply-all` | 스레드를 자동으로 연결해 답장 | `--message-id <ID>`, `--body`, `--draft` |
| `gws gmail +forward` | 메일 전달 | `--message-id <ID>`, `--to` |
| `gws gmail users drafts list` | 초안 목록 | `--params '{"userId":"me"}'` |
| `gws gmail users drafts send` | 기존 초안 발송. **명시적 승인 필요** | `--json '{"id":"<DRAFT_ID>"}'` |
| `gws gmail users messages list` | 메일 목록 (원시 API) | `--params '{"userId":"me","q":"is:unread"}'` |

예시 (리마인더 초안):

```bash
gws gmail +send --to "someone@example.com" --subject "Reminder: weekly report" \
  --body "The weekly report is due Friday 5pm." --draft
```

## 3. Drive

| 명령 | 용도 | 주요 옵션 |
|---|---|---|
| `gws drive +upload <file>` | 메타데이터를 자동으로 채워 파일 업로드 | `--parent <FOLDER_ID>` (`config/gws.json` 값), `--name` |
| `gws drive files list` | 파일 검색 및 목록 | `--params '{"q":"name contains '\''report'\''","pageSize":10}'` |
| `gws drive files get` | 파일 메타데이터 조회 | `--params '{"fileId":"<ID>"}'` |
| `gws drive files create` | 폴더나 파일 생성 | `--json '{"name":"<name>","mimeType":"application/vnd.google-apps.folder"}'` |
| `gws drive files copy` / `update` | 파일 복사, 이름 변경, 이동 | `--params '{"fileId":"<ID>"}'`, `--json` |
| `gws drive files export` | Docs/Sheets 파일 내보내기 (예: PDF) | `--params '{"fileId":"<ID>","mimeType":"application/pdf"}' --output <path>` |

예시 (기본 폴더에 보고서 업로드):

```bash
gws drive +upload workspace/outputs/report.md --parent "<FOLDER_ID>"
```

## 4. Docs

| 명령 | 용도 | 주요 옵션 |
|---|---|---|
| `gws docs documents create` | 문서 생성 | `--json '{"title":"<title>"}'` |
| `gws docs documents get` | 문서 읽기 | `--params '{"documentId":"<ID>"}'` |
| `gws docs +write` | 문서 끝에 일반 텍스트 추가 | `--document <ID> --text "<text>"` |
| `gws docs documents batchUpdate` | 내용 삽입, 치환, 서식 지정 | `--params '{"documentId":"<ID>"}' --json '{"requests":[...]}'` |

새 문서는 Drive 루트에 생성된다. 필요하면 `gws drive files update`(`addParents`)로 대상 폴더로 옮긴다.

## 5. Sheets

| 명령 | 용도 | 주요 옵션 |
|---|---|---|
| `gws sheets spreadsheets create` | 스프레드시트 생성 | `--json '{"properties":{"title":"<title>"}}'` |
| `gws sheets +read` | 범위 읽기 | `--spreadsheet <ID> --range 'Sheet1!A1:D10'` |
| `gws sheets +append` | 행 추가 | `--values "a,b,c"` 또는 `--json-values '[["a","b"],["c","d"]]'` |
| `gws sheets spreadsheets values update` | 셀 값 덮어쓰기 | `--params '{"spreadsheetId":"<ID>","range":"...","valueInputOption":"USER_ENTERED"}' --json '{"values":[[...]]}'` |

범위는 작은따옴표로 감싼다. bash에서는 `!`가 히스토리 확장으로 해석된다.

## 6. 공통 옵션과 출력 형식

| 옵션 | 용도 |
|---|---|
| `--params <JSON>` | URL 및 쿼리 파라미터 |
| `--json <JSON>` | 요청 본문 (POST, PATCH, PUT) |
| `--upload <PATH>` / `--upload-content-type <MIME>` | 미디어 콘텐츠 업로드 |
| `--output <PATH>` | 바이너리 응답을 파일로 저장 |
| `--format json\|table\|yaml\|csv` | 출력 형식 (기본값: `json`) |
| `--page-all`, `--page-limit <N>`, `--page-delay <MS>` | 자동 페이지 넘김 (NDJSON, 기본 최대 10페이지) |
| `--dry-run` | 실행하지 않고 보낼 요청만 표시 (헬퍼 명령) |

종료 코드:

| 코드 | 의미 |
|---|---|
| 0 | 성공 |
| 1 | API 오류 |
| 2 | 인증 오류 (인증 정보 없음 또는 무효) |
| 3 | 입력 검증 오류 (잘못된 인수) |
| 4 | Discovery 오류 (API 스키마를 가져오지 못함) |
| 5 | 내부 오류 |

셸 따옴표 처리: Git Bash에서는 예시를 그대로 쓴다. Windows PowerShell 5.1에서는 JSON 안의 큰따옴표를 이스케이프해야 할 수 있다 (`'{\"pageSize\": 5}'`).

## 7. 금지 및 제한 명령

| 명령 | 규칙 |
|---|---|
| `--draft` 없는 `gws gmail +send`, `gws gmail users drafts send` | 별도의 명시적 승인 후에만 실행 (`docs/approval-policy.md`) |
| `gws drive files delete`, `gws drive files emptyTrash` | 금지 (영구 삭제) |
| `gws gmail users drafts delete`, `gws gmail users messages delete` / `trash` | 사용자가 명시적으로 요청한 경우가 아니면 금지 |
| `gws drive permissions create` / `update` / `delete` | 명시적 승인 없이 공유 설정을 바꾸지 않는다 |
| `gws auth export` | 금지 (비밀 값 출력) |

승인된 쓰기 명령은 헬퍼가 지원하면 먼저 `--dry-run`으로 실행하고, 그 미리보기를 실행 계획에 포함한다.

## 8. 자주 발생하는 오류와 대응

| 증상 | 원인 | 대응 |
|---|---|---|
| 종료 코드 2 / `invalid_grant` | 토큰 만료 (테스트 모드 7일 제한) 또는 취소 | `gws auth login -s gmail,drive,docs,sheets` |
| 브라우저에서 `403 access_denied` | 계정이 OAuth 앱의 테스트 사용자가 아님 | Google 인증 플랫폼 → 대상 → 테스트 사용자에 계정 추가 |
| `accessNotConfigured` (403) | GCP 프로젝트에서 API가 꺼져 있음 | `gcloud services enable <api>.googleapis.com --project <id>` 실행 후 10초쯤 기다렸다가 재시도 |
| `insufficient scopes` (403) | 로그인할 때 해당 범위를 받지 않음 | 빠진 범위를 넣어 다시 로그인 (`-s ...,<service>`) |
| `redirect_uri_mismatch` | OAuth 클라이언트 유형이 Desktop app이 아님 | Desktop app 유형으로 OAuth 클라이언트를 새로 생성 |
| 종료 코드 3 | 잘못된 JSON이나 인수 | 따옴표 처리 확인, `gws schema <method>`로 파라미터 확인 |
| 종료 코드 4 | API 스키마를 가져오지 못함 | 네트워크 확인 후 재시도 |
